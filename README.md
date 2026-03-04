# Introduction
This repository is based on Unitree's repositories and aims to add the compatibility of the ROS packages with ROS Noetic. The list of the repositories considered is the following:
* [unitree_ros](https://github.com/unitreerobotics/unitree_ros)
* [unitree_ros_to_real](https://github.com/unitreerobotics/unitree_ros_to_real) (only the package `unitree_legged_msgs`)
* [unitree_guide](https://github.com/unitreerobotics/unitree_guide)

The repository contains all the necessary packages to run a simulation with Unitree robots as well as with real robot. You can load robots and joint controllers in Gazebo and you can move the robot in the environment. There are also packages that enable SLAM and navigation functionalities.

# Dependencies
* [ROS Noetic](https://www.ros.org/)
* [Gazebo](http://gazebosim.org/)

# Build

For ROS Noetic:
```
sudo apt-get update
sudo apt-get install liblcm-dev
sudo apt-get install ros-noetic-controller-interface ros-noetic-gazebo-ros-pkgs ros-noetic-gazebo-ros-control ros-noetic-joint-state-controller ros-noetic-effort-controllers ros-noetic-joint-trajectory-controller ros-noetic-amcl ros-noetic-move-base ros-noetic-slam-gmapping ros-noetic-hector-slam ros-noetic-map-server ros-noetic-global-planner ros-noetic-dwa-local-planner ros-noetic-rtabmap-ros ros-noetic-realsense2-camera ros-noetic-realsense2-description
```

Clone this repository in your catkin workspace:
```
git clone https://github.com/Sgt-Hashtag/Unitree-A1-Real-Ros
```
Update the submodules:
```
git submodule init
git submodule update --recursive
```
And open the file `unitree_ros/unitree_gazebo/worlds/stairs.world`. At the end of the file (line 112):
```
<include>
    <uri>model:///home/unitree/catkin_ws/src/ros_unitree/unitree_ros/unitree_gazebo/worlds/building_editor_models/stairs</uri>
</include>
```
Please change the path of `building_editor_models/stairs` to the real path on your PC.

Then you can use `catkin_make` to build:
```
cd ~/catkin_ws
catkin_make
```

If you face a dependency problem, you can just run `catkin_make` again.
Here the CMakeLists for unitree_guide is set to Real robot by default but you can change it Simulation depending upon what is needed. But you cannot use Both, since the build latches to either the Gazebo or Real robot ros topics for Rviz interface.

# Robots Description
The description of robots Go1, A1, Aliengo, and Laikago. Each package includes mesh, urdf and xacro files of robot. Take Go1 for example, you can check the model in Rviz by:
```
roslaunch a1_description a1_rviz.launch
```
![A1 in Rviz](./src/ros_unitree/doc/unitree_a1_rviz.png)

# Start the Simulation
Open a terminal and start Gazebo with a preloaded world:
```
roslaunch unitree_gazebo robot_simulation.launch rname:=a1 wname:=earth
```
Where the `rname` means robot name, which can be `laikago`, `aliengo`, `a1` or `go1`. The `wname` means world name, which can be `earth`, `space` or `stairs`. And the default value of `rname` is `go1`, while the default value of `wname` is `earth`. In Gazebo, the robot should be lying on the ground with joints not activated.

![A1 in Passive state](./src/ros_unitree/doc/unitree_a1_gazebo_1.png)

For starting the controller, open a new terminal, then run the following command:
```
rosrun unitree_guide base_ctrl
```
After starting the controller, the robot will lie on the ground of the simulator, then press the '2' key on the keyboard to switch the robot's finite state machine (FSM) from Passive(initial state) to FixedStand, then press the '4' key to switch the FSM from FixedStand to Trotting

![A1 in Trotting state](./src/ros_unitree/doc/unitree_a1_gazebo_2.png)

Now you can press the 'W', 'A', 'S', 'D' keys to control the translation of the robot, and press the 'J', 'L' keys to control the rotation of the robot. Press the 'Spacebar', the robot will stop and stand on the ground. If there is no response, you need to click on the terminal opened for starting the controller and then repeat the previous operation.

![A1 moving](./src/ros_unitree/doc/unitree_a1_gazebo_3.gif)

The full list of transitions between states available is the following:
* Key '1': FixedStand/FreeStand to Passive
* Key '2': Passive/Trotting to FixedStand
* Key '3': Fixed stand to FreeStand
* Key '4': FixedStand/FreeStand to Trotting
* Key '5': FixedStand to MoveBase
* Key '8': FixedStand to StepTest
* Key '9': FixedStand to SwingTest
* Key '0': FixedStand to BalanceTest

# Gazebo Worlds

The worlds included in this repository are simple worlds and they should be used for dedug purposes. If you want to add more realistc worlds, you can checkout the repository [gazebo_worlds](https://github.com/leonhartyao/gazebo_models_worlds_collection).
After following the instructions for the usage, you can specify which world you want to load with the argument `wname`:

```
roslaunch unitree_gazebo robot_simulation.launch rname:=a1 wname:=office_small
```

![World loaded in Gazebo](./src/ros_unitree/doc/unitree_a1_gazebo_world.png)

# Start Realsense in Real robot
On your pc:
export ROS_MASTER_IP=http://192.168.123.161:11311
export ROS_IP=192.168.123.114

Then for Intel PC over ssh:
ssh -X unitree@192.168.123.161
```
roslaunch unitree_real real.launch
```
2nd Terminal
```
rosrun unitree_guide base_ctrl
```
Then for Nvidia PC over ssh:
ssh -X unitree@192.168.123.12
```
roslaunch realsense2_camera rs_camera.launch depth_width:=424 depth_height:=240 depth_fps:=30 color_width:=424 color_height:=240 color_fps:=30 enable_pointcloud:=true align_depth:=true filters:=decimation decimation_filter_magnitude:=2 image_compression:=true
```

# Mapping

Start the simulation in Gazebo:
```
roslaunch unitree_gazebo robot_simulation.launch rname:=a1 wname:=office_small rviz:=false
```
Start the pcl2lasescan node:
```
roslaunch pcl2scan pcl2scan.launch rname:=a1 use_sim:=true
```

Start the robot controller:
```
rosrun unitree_guide base_ctrl
```


##NORMAL MAP


Start the mapping process with gmapping:
```
roslaunch unitree_navigation slam.launch rname:=a1 rviz:=true algorithm:=gmapping
```
Alternatively, you can start the mapping with hector:
```
roslaunch unitree_navigation slam.launch rname:=a1 rviz:=true algorithm:=hector
```
In the terminal of the robot controller, press the keys '2' and '4' to set the robot in trotting mode, then, move the robot around the environment to create the map.

During the mapping process, Rviz should show something similar to the following screehshot:
![a1 mapping in Rviz](./src/ros_unitree/doc/unitree_a1_mapping_rviz.png)

Save the map:
```
rosrun map_server map_saver -f ~/catkin_ws/src/ros_unitree/unitree_guide/unitree_navigation/maps/office_small
```


##OCTO MAP
start mapping with configured octomap package
```
sudo apt-get update
sudo apt-get install ros-noetic-octomap-ros ros-noetic-octomap-msgs ros-noetic-octomap-server
```

```
roslaunch mapping3d_realsense mapping3d.launch  use_sim:=true
```
![a1 octomap in Rviz](./src/ros_unitree/doc/unitree_a1_octomap.png)

save octomap created
```
rosrun octomap_server octomap_saver -f small_office_octopmap.bt /ot

```
Use MarkerArray from Rviz


# Navigation
Start the simulation in Gazebo:
```
roslaunch unitree_gazebo robot_simulation.launch rname:=a1 wname:=office_small rviz:=false
```

Start the robot controller:
```
rosrun unitree_guide base_ctrl
```
and press the keys '2' and '5' to activate the MoveBase mode.

Start the pcl2lasescan node:
```
roslaunch pcl2scan pcl2scan.launch rname:=a1
```

Start the navigation stack:
```
roslaunch unitree_navigation navigation.launch rname:=a1 map_file:=/home/isaac/Documents/catkin_ws/src/ros_unitree/unitree_guide/unitree_navigation/maps/office_small.yaml
```


Alternatively publish octomap:
```
rosrun octomap_server octomap_server_node src/ros_unitree/mapping3d_realsense/maps/small_office_octopmap.bt
```
Use MarkerArray from Rviz

In Rviz, first set the initial position of the robot with the "2D Pose Estimate" button. Next, set a navigation goal with the "2D Nav Goal" button.


"For static transform"
rosrun tf static_transform_publisher 0 0 0 0 0 0 1 map odom 100
![A1 navigation](./src/ros_unitree/doc/unitree_a1_navigation.gif)

# Note
If you are having problems with the movements of the robot and the node base_ctrl logs the following error:
```
[ERROR] Function setProcessScheduler failed.
```
Yo should consider to edit the file `/etc/security/limits.conf` and add the following two lines:
```
<username> hard rtprio 99
<username> soft rtprio 99
```
Replace `<username>` with your username.
Save, close the file and reboot the system to apply the changes.

---
## fixes
```
#include <cstdint> Add this at the top of: grid_map/grid_map_core/include/grid_map_core/TypeDefs.hpp
#include <array>  Add this at the top of: grid_map_sdf/include/grid_map_sdf/SignedDistanceField.hpp
```


Replace /UnitreeA1/Unitree_ws/src/grid_map/grid_map_pcl/Cmakelists.txt with the following:
```
cmake_minimum_required(VERSION 3.5.1)
project(grid_map_pcl)

set(CMAKE_CXX_STANDARD 17)
add_compile_options(-Wall -Wextra -Wpedantic)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(SRC_FILES
  src/GridMapPclConverter.cpp
  src/GridMapPclLoader.cpp
  src/helpers.cpp
  src/PclLoaderParameters.cpp
  src/PointcloudProcessor.cpp
)

set(CATKIN_PACKAGE_DEPENDENCIES  
  grid_map_core
  grid_map_msgs
  grid_map_ros
  pcl_ros
  roscpp
)

find_package(OpenMP QUIET)
if (OpenMP_FOUND)
  add_compile_options("${OpenMP_CXX_FLAGS}")
  add_definitions(-DGRID_MAP_PCL_OPENMP_FOUND=${OpenMP_FOUND})
endif()

find_package(PCL REQUIRED
  COMPONENTS
    common
    features
    filters
    io
    kdtree
    segmentation
    surface
)

## Find catkin macros and libraries
find_package(catkin REQUIRED 
  COMPONENTS
    ${CATKIN_PACKAGE_DEPENDENCIES}
)

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
  INCLUDE_DIRS
    include
  LIBRARIES
    ${PROJECT_NAME}
  CATKIN_DEPENDS
    ${CATKIN_PACKAGE_DEPENDENCIES}
  DEPENDS
    PCL
)

###########
## Build ##
###########

# Library.
add_library(${PROJECT_NAME}
  ${SRC_FILES}
)
add_dependencies(${PROJECT_NAME}
  ${catkin_EXPORTED_TARGETS}
)
target_include_directories(${PROJECT_NAME} PRIVATE
  include
)
target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${OpenMP_CXX_INCLUDE_DIRS}
  ${PCL_COMMON_INCLUDE_DIRS}
  ${PCL_FEATURES_INCLUDE_DIRS}
  ${PCL_FILTERS_INCLUDE_DIRS}
  ${PCL_IO_INCLUDE_DIRS}
  ${PCL_KDTREE_INCLUDE_DIRS}
  ${PCL_SEGMENTATION_INCLUDE_DIRS}
  ${PCL_SURFACE_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${OpenMP_CXX_LIBRARIES}
  yaml-cpp::yaml-cpp
)


# Nodes.
add_executable(grid_map_pcl_loader_node
  src/grid_map_pcl_loader_node.cpp
)
add_dependencies(grid_map_pcl_loader_node
  ${PROJECT_NAME}
)
target_include_directories(grid_map_pcl_loader_node PRIVATE
  include
)
target_include_directories(grid_map_pcl_loader_node SYSTEM PUBLIC
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)
target_link_libraries(grid_map_pcl_loader_node
  ${PROJECT_NAME}
  ${catkin_LIBRARIES}
)

add_executable(pointcloud_publisher_node
  src/pointcloud_publisher_node.cpp
)
add_dependencies(pointcloud_publisher_node
  ${PROJECT_NAME}
)
target_include_directories(pointcloud_publisher_node PRIVATE
  include
)
target_include_directories(pointcloud_publisher_node SYSTEM PUBLIC
  ${catkin_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)
target_link_libraries(pointcloud_publisher_node
  ${PROJECT_NAME}
  ${catkin_LIBRARIES}
)

#############
## Install ##
#############
install(
  TARGETS
    ${PROJECT_NAME}
    grid_map_pcl_loader_node
    pointcloud_publisher_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
install(
  DIRECTORY 
    include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.hpp"
)
install(
  DIRECTORY
    doc
    config
    launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(
  DIRECTORY
    test
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  FILES_MATCHING PATTERN "*.pcd"
)
install(
  FILES
    README.md
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)


#############
## Testing ##
#############

if(CATKIN_ENABLE_TESTING)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")

  find_package(catkin REQUIRED
    COMPONENTS
      ${CATKIN_PACKAGE_DEPENDENCIES}
      roslaunch
  )
  find_package(yaml-cpp REQUIRED)

  roslaunch_add_file_check(launch)

  ## Add gtest based cpp test target and link libraries
  catkin_add_gtest(${PROJECT_NAME}-test
    test/test_grid_map_pcl.cpp
    test/GridMapPclLoaderTest.cpp
    test/HelpersTest.cpp
    test/PointcloudProcessorTest.cpp
    test/test_helpers.cpp
    test/PointcloudCreator.cpp
  )
  target_include_directories(${PROJECT_NAME}-test PRIVATE
    include
  )
  target_include_directories(${PROJECT_NAME}-test SYSTEM PUBLIC
    ${catkin_INCLUDE_DIRS}
    ${EIGEN3_INCLUDE_DIR}
    ${OpenMP_CXX_INCLUDE_DIRS}
  )
  target_link_libraries(${PROJECT_NAME}-test 
    ${PROJECT_NAME}
    ${catkin_LIBRARIES}
  )

  ###################
  ## Code_coverage ##
  ###################
  find_package(cmake_code_coverage QUIET)
  if(cmake_code_coverage_FOUND)
    add_gtest_coverage(
      TEST_BUILD_TARGETS
        ${PROJECT_NAME}-test
    )
  endif()
endif()

#################
## Clang_tools ##
#################
find_package(cmake_clang_tools QUIET)
```
