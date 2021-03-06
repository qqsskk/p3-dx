[Table of Contents](#table-of-contents)

[Introduction](#introduction)

[Dependencies](#dependencies)

[Bring up the Robot](#bring-up-the-robot)
-	[Stream Sonar Point Clouds](#stream-sonar-point-clouds)
-	[Stream Laser 2D clouds (sensor_msgs/PointCloud)](#stream-laser-2D-clouds-(sensor_msgs/pointCloud))
-	[Stream Laser 3D Point clouds](#stream-laser-3d-point-clouds)
-	[Publish dynamic tf transform Twist messages to robot](#publish-dynamic-tf-transform-twist-messages-to-robot)
[Running the Navigation Stack](#running-the-navigation-stack)
-	[Start all the nodes above simultaneously](#start-all-the-nodes-above-simultaneously)
-	[Navigate the environment using ROS' navigation stack](#Navigate-the-environment-using-ROS'-navigation-stack)

[Questions](#questions)


###Introduction

This repo has a set of clients that, among other things, pushes velocity commands to the P3_DX robot from [adept mobile robots](http://www.mobilerobots.com/ResearchRobots/PioneerP3DX.aspx), displays the point clouds from the scan results of the p3_dx LIDAR sensor and SONAR. The package supports indoor localization and dynamic SLAM via an adaptive Monte Carlo Localization (AMCL) for mobile robots as described by Sebastian Thrun, Wolfram Burgard, and Dieter Fox in their book, <i>Probabilistic Robotics, Intelligent Robotics and Autonomous Agent</i>. 

To aid faster implementation time, we have developed the code in ROS. In addition to sending velocity commands, and performing dynamic SLAM based on LIDAR data, it subscribes to the [RosAria package's](wiki.ros.org/rosaria) sonar scans, laser scans, and projected 3D laser scans(point clouds) and provides a <i>400 X 400</i> pixels window to visualize these topics in real-time. Example point clouds from the sonars and laser scanners are provided below:

<div class="fig figcenter fighighlight">
	<a href="" target="_blank">
		<img src="http://www.mobilerobots.com/Libraries/Site_Images/P3-DXwith_ball_2.sflb.ashx" height="250px" width="250px" align="left" hspace="40"></a>
	<a href="https://youtu.be/lYgp8qZjvks" target="_blank">
		<img src="/p3dx_2dnav/map_data/laser3d.png" height="250px" width="250px" alight="right"></a>
	<div class="figcaption" align="left" hspace="10">P3-DX Robot</div>
	<div class="figcaption" align="middle">Laser 3D Point Clouds</div>
</div>

<br></br>
<div class="fig figcenter fighighlight">
	<a href="https://youtu.be/B871f3qa1p4" target="_blank">
		<img src="/p3dx_2dnav/map_data/laser2d.png" height="250px" width="250px" align="left" hspace="40"></a>
	<a href="https://youtu.be/PYT4FCIVYgw" target="_blank">
		<img src="/p3dx_2dnav/map_data/sonar3d.png" height="250px" width="250px" alight="right"></a>
	<div class="figcaption" align="left" hspace="10">Laser 2D Point Clouds</div>
	<div class="figcaption" align="middle">Sonar 3D Point Clouds</div>
</div>
<!-- 
[Aria](http://www.mobilerobots.com/Software/ARIA.aspx) package and [Arnl](http://www.mobilerobots.com/Software/NavigationSoftware.aspx) package from Adept.  -->

Here is an example video of the navigation of the robot based on velocity commands that are sent to the robot after receiving the `TF` transform broadcasters from the `rosaria` package:

<div class="fig figcenter fighighlight">
<a href="https://youtu.be/yczG8CUbK2M">
	<img src="https://i.ytimg.com/vi/yczG8CUbK2M/1.jpg?time=1466972319359" height="300px" width="400px" align="middle" hspace="100"></a>
	</br>
	<div class="figcaption" align="left" hspace="80">Twist commands to the p3_dx robot</div>
</div>

To compile these codes, you would want to pull the files from the links indicated above. 

### Dependencies

Please install the [Aria](http://robots.mobilerobots.com/wiki/ARIA#Download_Aria) and [Arnl](http://robots.mobilerobots.com/wiki/ARNL,_SONARNL_and_MOGS#Download) packages by following the intructions on the links, [Aria](http://www.mobilerobots.com/Software/ARIA.aspx) package and [Arnl](http://www.mobilerobots.com/Software/NavigationSoftware.aspx). Also, you would want to install the ros wrappers to the Aria and Arnl packages namely: [rosarnl](https://github.com/MobileRobots/ros-arnl) and [rosaria](http://wiki.ros.org/ROSARIA). 

In addition to the above dependencies, you would preferrably want to compile the code using c++11. On Linux, ensure you have at least g++ 4.8 and pass the `-std=c++11` to the CMakeLists.txt files (this is already done by default in the accompanying CMakeLists file).

When you are all set, you can clone this repo to your catkin workspace `src` folder and build with 

```bash
	catkin_make --source src/p3_dx
```
### Bring up the Robot

When the compilation finishes, you can run each individual executable as follows:

#### 1. Stream Sonar Point Clouds

```bash
	rosrun sonar_clouds sonar_clouds
```

Remember to click on the PCL window and zoom out on the clouds for visibility.

#### 2. Stream Laser 2D clouds (sensor_msgs/PointCloud)

```bash
	rosrun laser_scans laser_scans
```

#### 3. Stream Laser 3D Point clouds

```bash
	rosrun scanner_clouds scanner_clouds
```

#### 4. Publish dynamic `tf transform` Twist messages to the `RosAria` topic: `/RosAria/cmd_vel`

After retrieving RosAria's latest published transforms, from the `odometry` frame -> `base_link` -> `laser frame`, we generate the transform from the origin to a new pose at time `t_1` and we move linearly along x according to the following relation:

```lua
vel_msg.linear.x = 0.5 * sqrt(pow(transform.getOrigin().x(), 2) +
                              pow(transform.getOrigin().y(), 2));
``` 
and orient the robot along `z` according to:

```lua
vel_msg.angular.z = 4.0 * atan2(transform.getOrigin().y(),
                                transform.getOrigin().x());

```
To start the lookuptransform, do this in a separate terminal:

```bash
	rosrun tf_listener tf_listener
```

To generate dynamic motions based on the relations above, pass `-p` or `-pirouette` to the command above.


### Running the Navigation Stack

#### 1. Terminal 1: Start all the nodes above simultaneously

```bash
	roslaunch p3dx_2dnav p3_dx.launch
```

#### 2. Terminal 2: Navigate the environment using ROS' navigation stack 
 This uses the adaptive monte carlo localization algorithm as thoroughly discussed by Dieter Fox, Thrun, and colleagues in their book, <i>Probabilistic Robotics.</i> Fire up a separate terminal and do:

```bash
	roslaunch p3dx_2dnav move_base.launch
```

A static map of the environment (generated with [openslam's gmapping](http://openslam.org/gmapping.html)) that was used for development is provided in the directory [p3dx_2dnav/map_data/map.pgm](/p3dx_2dnav/map/map.pgm). Feel free to create your own map and feed it to the robot by following the [Ros Map Server Tutorial](http://wiki.ros.org/map_server). Also provided along with the map are the global costmap, local costmap, base local planner and costmap common parameters that are used in setting up the ROS navigation stack when you bring up the robot 

 This will navigate the environment with all the robot's sensors and will dynamically update map of the environment based on real-time acquired sensor information. 


### Questions

Please use the issues page.

