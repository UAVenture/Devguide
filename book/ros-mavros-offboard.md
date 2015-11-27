# MAVROS offboard control example

<aside class="caution">
Offboard control is dangerous. If you are operating on a real vehicle be sure to have a way of gaining back manual control in case something goes wrong.
</aside>

The following tutorial will run through the basics of offboard control through mavros as applied to an Iris quadcopter simulated in Gazebo. At the end of the tutorial, you should see the same behaviour as in the video below, i.e. a slow takeoff to an altitude of 2 meters.

<video width="100%" autoplay="true" controls="true">
	<source src="images/sim/gazebo_offboard.webm" type="video/webm">
</video>

## Code
Create the offb_node.cpp file in your ros package and paste the following inside it:
```C++
/**
 * @file offb_node.cpp
 * @brief offboard example node, written with mavros version 0.14.2, px4 flight
 * stack and tested in Gazebo SITL
 */

#include <ros/ros.h>
#include <geometry_msgs/PoseStamped.h>
#include <mavros_msgs/CommandBool.h>
#include <mavros_msgs/SetMode.h>

int main(int argc, char **argv)
{
    ros::init(argc, argv, "offb_node");
    ros::NodeHandle nh("~");

    ros::Publisher local_pos_pub = nh.advertise<geometry_msgs::PoseStamped>
            ("/mavros/setpoint_position/local", 1);
    ros::ServiceClient arming_client = nh.serviceClient<mavros_msgs::CommandBool>
            ("/mavros/cmd/arming");
    ros::ServiceClient set_mode_client = nh.serviceClient<mavros_msgs::SetMode>
            ("/mavros/set_mode");

    mavros_msgs::SetMode offb_set_mode;
    offb_set_mode.request.custom_mode = "OFFBOARD";

    geometry_msgs::PoseStamped pose;
    pose.pose.position.x = 0;
    pose.pose.position.y = 0;
    pose.pose.position.z = 2;

    //the setpoint publishing rate MUST be faster than 2Hz
    ros::Rate rate(20.0);

    int i = 100;
    while(ros::ok() && i > 0){
        local_pos_pub.publish(pose);
        i--;
        ros::spinOnce();
        rate.sleep();
    }

    mavros_msgs::CommandBool arm_cmd;
    arm_cmd.request.value = true;

    bool offboard_enabled = false;
    bool armed = false;

    while(ros::ok()){
        if(!offboard_enabled){
            // The mode switch should be accepted once a few setpoints are sent
            if(set_mode_client.call(offb_set_mode) &&
                    offb_set_mode.response.success){
                ROS_INFO("Offboard enabled");
                offboard_enabled = true;
            }
        } else {
            if(!armed){
                if(arming_client.call(arm_cmd) &&
                        arm_cmd.response.success){
                    ROS_INFO("Vehicle armed");
                    armed = true;
                }
            }
        }

        local_pos_pub.publish(pose);

        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}

```
## Code explanation
```C++
#include <ros/ros.h>
#include <geometry_msgs/PoseStamped.h>
#include <mavros_msgs/CommandBool.h>
#include <mavros_msgs/SetMode.h>
```
The `mavros_msgs` package contains all of the custom messages required to operate services and topics provided by the mavros package. All services and topics as well as their corresponding message types are documented in the [mavros wiki](http://wiki.ros.org/mavros).

```C++
ros::Publisher local_pos_pub = nh.advertise<geometry_msgs::PoseStamped>("/mavros/setpoint_position/local", 1);
ros::ServiceClient arming_client = nh.serviceClient<mavros_msgs::CommandBool>("/mavros/cmd/arming");
ros::ServiceClient set_mode_client = nh.serviceClient<mavros_msgs::SetMode>("/mavros/set_mode");
```
We instantiate a publisher to publish the commanded local position and the appropriate clients to request arming and mode change.

```C++
mavros_msgs::SetMode offb_set_mode;
offb_set_mode.request.custom_mode = "OFFBOARD";
```
We set the custom mode to `OFFBOARD`. A list of [supported modes](http://wiki.ros.org/mavros/CustomModes#PX4_native_flight_stack) is available for reference.

```C++
geometry_msgs::PoseStamped pose;
pose.pose.position.x = 0;
pose.pose.position.y = 0;
pose.pose.position.z = 2;
```
Even though the px4 flight stack operates in the aerospace NED coordinate frame, mavros translates these coordinates to the standard ENU frame and vice-versa. This is why we set z to positive 2.

```C++
//the setpoint publishing rate MUST be faster than 2Hz
ros::Rate rate(20.0);
```
The px4 flight stack has a timeout of 500ms between two offboard commands. If this timeout is exceeded, the commander will fall back to the last mode the vehicle was in before entering offboard mode. This is why the publishing rate **must** be faster than 2 Hz to also account for possible latencies. This is also the same reason why it is recommended to enter offboard mode from POSCTL mode, this way if the vehicle drops out of offboard mode it will stop in its tracks and hover.

```C++
int i = 100;
while(ros::ok() && i > 0){
    local_pos_pub.publish(pose);
    i--;
    ros::spinOnce();
    rate.sleep();
}
``` 
Before entering offboard mode, you must have already started streaming setpoints otherwise the mode switch will be rejected. Here, 100 was chosen as an arbitrary amount.

```
while(ros::ok()){
    if(!offboard_enabled){
        // The mode switch should be accepted once a few setpoints are sent
        if(set_mode_client.call(offb_set_mode) &&
                offb_set_mode.response.success){
            ROS_INFO("Offboard enabled");
            offboard_enabled = true;
        }
    } else {
        if(!armed){
            if(arming_client.call(arm_cmd) &&
                    arm_cmd.response.success){
                ROS_INFO("Vehicle armed");
                armed = true;
            }
        }
    }

    local_pos_pub.publish(pose);

    ros::spinOnce();
    rate.sleep();
}
```
The rest of the code is pretty self explanatory. We attempt to switch to offboard mode after which we arm the quad to allow it to fly. In the same loop we continue sending the requested pose at the appropriate rate.