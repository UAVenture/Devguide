# Gazebo Simulation

[Gazebo](http://gazebosim.org) is a 3D simulation environment for autonomous robots. It can be used with [ROS as complete robotics simulation suite](https://pixhawk.org/dev/ros/sitl) or standalone, and this guide covers the simpler to set up standalone operation.

{% youtube %}https://www.youtube.com/watch?v=qfFF9-0k4KA&vq=hd720{% endyoutube %}


```mermaid
graph LR;
  Gazebo-->Plugin;
  Plugin-->MAVLink;
  MAVLink-->SITL;
```

## Installation

The installation requires to install Gazebo and our simulation plugin.

<aside class="tip">
Mac OS users should install gazebo 7. Linux users: If you installed a ROS version earlier than Jade, be sure to uninstall the bundled Gazebo (sudo apt-get remove ros-indigo-gazebo) version as it is too old. Linux users should install gazebo 6.
</aside>

### Mac OS

Mac OS requires Gazebo 7.

<div class="host-code"></div>

```sh
brew install graphviz sdformat3 protobuf eigen opencv
brew install gazebo7
```

### Linux

[Linux installation instructions](http://gazebosim.org/tutorials?tut=install_ubuntu&ver=6.0&cat=install) for Gazebo 6.

Additionally needed:

```sh
sudo apt-get install libeigen3-dev protobuf-compiler
```

## Running the Simulation

Run the PX4 SITL with the Iris configuration in the Firmware directory:

<div class="host-code"></div>

```sh
cd ~/src/Firmware
make posix_sitl_default gazebo
```

This will bring up the PX4 shell:

```sh
[init] shell id: 140735313310464
[init] task name: mainapp

______  __   __    ___ 
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

Ready to fly.


pxh>
```

<aside class="note">
Right-clicking the quadrotor model allows to enable follow mode from the context menu, which is handy to keep it in view.
</aside>

## Taking it to the Sky

![](images/sim/gazebo.png)

The system will print the home position once it finished intializing (`telem> home: 55.7533950, 37.6254270, -0.00`). You can bring it into the air by typing:

```sh
pxh> commander takeoff
```

<aside class="tip">
Joystick or thumb-joystick support is available through QGroundControl (QGC). To use manual input, put the system in a manual flight mode (e.g. POSCTL, position control). Enable the thumb joystick from the QGC preferences menu.
</aside>

## Extending and Customizing

To extend or customize the simulation interface, edit the files in the `Tools/sitl_gazebo` folder. The code can be accessed through the [sitl_gazebo repository](https://github.com/px4/sitl_gazebo) on Github.

<aside class="note">
The build system enforces the correct submodule to be checked out for all dependencies, including the simulator. It will not overwrite changes in files in the directory, however, when these changes are comitted the submodule needs to be registered in the Firmware repo with the new commit hash. To do so, `git add Tools/sitl_gazebo` and commit the change. This will update the GIT hash of the simulator.
</aside>

## Interfacing to ROS

The simulation can be [interfaced to ROS](simulation-ros-interface.md) the same way as onboard a real vehicle.
