# Autonomous-Mobile-Robot
Prototype development of Modular Autonomous Mobile Robot with interchangeable extensions to enhance versatility. The primary focus of the Autonomous Mobile Robot (AMR) is targeted at point-to-point applications within warehouse settings. Specifically, it helps in transporting materials from one station to another as needed. The utilization of LiDAR sensors ensures precise robot localization and navigation, enhancing the efficiency of the entire system.

![AMR1](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/87f42b2d-acdd-47a4-a349-afde333b44a4)


## Prerequisites
- Ubuntu 18.04.6 LTS (Bionic Beaver)
- Robot Operating System: Melodic
- Python 3.8 or higher
- C++ 11 or higher


# Drive Wheel System

## Building the Chassis

1. Disassembled the Hoverboard to extract the Hoverboard wheels, hub motors and the hoverboard programmer.
2. Cut aluminum extrusions according to the required size to build the chassis.

![AMR3](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/35a26c1f-3eb0-440f-960e-ce9261b187be)


## Building the Drive Wheel System

1. Flashed the hoverboard programmer using the STM Link 32. This was done so that , we could tailor the programmer according to our AMR application. A hex file was generated using the Platform IO by flashing the Hoverboard Firmware Hack FOC where a USART communication was established between the hoverboard programmer and the Jetson Nano.

![AMR4](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/7dbe8f41-99ca-408d-94cc-c9c264dc89bb)

3. Connection beteween the Jetson nano and the programmer and running ROS on the microcomputer The TX-RX and the GND was connected to the USART pins and the GPIO pins of the Jetson. ROS -melodic was installed and tele-op operations were run using the Hoverboard-driver ROS package. Refered : https://github.com/alex-makarov/hoverboard-driver The ROS package was installed on the Jetson and the parameters were changed to the pins connected. The Teleop keyboard package was launched and the hoverboard wheels were controlled using the keyboard. The max_jerk_limit was defined to zero to stop the jerking effect while controlling the wheels.

![Screenshot 2024-02-04 164004](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/f89781d4-afc7-4a30-9221-c27f505fa735)


# Simulation

1. Designed the Robot model in Autdesk Fusion 360 and converted it to the URDF (Universal Robot Description Format ) file which was to be used in Gazebo for simulation.
2. A simulated map was also created in Fusion 360 and Blender where the Robot model was spawned and mapped.

![AMR6](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/56548ce8-39dd-485b-b131-9436d548b3d0)

3. The Xacro file in the URDF was changed and plug-ins for the LiDARs and Differential Drive were added.
4. The designed map was reopened in Gazebo and was saved as the .world file and changed in gazebo.launch file were made.
5. The Mobile robot was then spawned in the map and the mapping was performed using the Lidars using Gmapping node, thus implementing SLAM in ROS.
   1. It is a necessary package for the autonomous navigation of the mobile robot. It provides laser-based SLAM as a ros node called "slam_gmapping"

![AMR7](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/74e08653-a3ec-4db3-8359-da944c70a76f)

6. R-Viz was utilized as the visualization software and the map visualized by the Lidars was seen in the environment.
   1. The ROS navigation stack is a collection of software packages that you can use to help your robot move from a starting location to the end location safely.
7. We needed to make sure that there was no error in the Robot Mdoel and the topic /scan data in R-Viz.
8. After the map was saved by scanning the Lidar data in Gazrbo, autonomous navigation was performed using the AMCL package. The AMCL package is Adaptive Monte-Carlo Localization algorithm which uses a probablistic approch for localizing the robot and navigating within a known map.
   1. AMCL is a two dimensional probabilistic localization system. It uses a particle filter to track a robot's pose against a known map.
   2. However, in this project, the AMCL package was used only for simulation. For the actual robot, Cartographer was implemented. Cartographer takes data from LiDAR laser scans to estimate the positions of the robot.

 ![AMR5](https://github.com/AabidPatel/Modular-Autonomous-Mobile-Robot/assets/73630123/153c4e2c-9632-44d0-82cc-648ab5304f82)


 # Autonomous Navigation

## Once the simulation runs properly without any errors and the robot assembly is done we can execute the following commands to operate the robot autonomously.

1. We needed to first provide permission to each of the ports on the Jetson for the 2-Lidars and the hoverboard driver using the following cmd:
```
sudo chmod 666 /dev/ttyTHS1
sudo chmod 777 /dev/ttyUSB0
sudo chmod 777 /dev/ttyUSB1
```
2. The real-time data of the lidars were computed using the YD_lidar ROS package. To run the front and the back lidar we used the following cmd:
```
roslaunch tortoisebot_firmware fl.launch
roslaunch tortoisebot_firmware bl.launch
```
3. Since we used two lidars we merged their data together using the IRA_tools_merger and adjusting their TF-models in RViz.The /front_scan and /back_scan data from both the Lidars were merged to the topic /scan data by publishing data to virtual lidars.
```
roslaunch ira_laser_tools merge.launch
```
4. The Mapping of the robot in the environment was done in R-Viz by utilizing the Gmapping node and Cartographer. Using the Google Cartographer algorithm eliminated the use of encoders to localize the robot in the known map.
```
rosrun teleop_twist_keyboard teleop_twist_keyboard.py cmd_vel:=/hoverboard_velocity_controller/cmd_vel
roslaunch tortoisebot_firmware bringup.launch
roslaunch tortoisebot_firmware server_bringup.launch
roslaunch tortoisebot_slam tortoisebot_slam.launch
```
5. Autonomous Navigation: The navigation stack was implemented on the robot and parameters according to the size of the actual robot were defined.
   1. The Global and Local cost-map parameters were defined for the AMR to know its restrictions for navigating in the map and move from one point to another.
   2. The value of sim-time was changed for adjusting the speed of the robot to reach its destination.
   3. The target-position was defined and the robot was navigating by detecting the static and dynamic obstcles in the map to the final destination.
   4. We launch the scanned map and then launch the navigation stack:
```
rosrun map_server map_saver -f home_1
roslaunch tortoisebot_navigation tortoisebot_navigation.launch
```
