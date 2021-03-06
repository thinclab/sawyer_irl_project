cmake_minimum_required(VERSION 2.8.3)
project(sawyer_irl_project)

## Compile as C++11, supported in ROS Kinetic and newer
add_compile_options(-std=c++11)
add_definitions(-std=c++11)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  tf2_geometry_msgs
  moveit_ros_planning_interface
  controller_manager
  message_generation
  robotiq_2f_gripper_control
)
find_package(gazebo REQUIRED)
link_directories(${GAZEBO_LIBRARY_DIRS})
find_package(Boost REQUIRED COMPONENTS system)

add_message_files(
  FILES
  onions_blocks_poses.msg
  OBlobs.msg
  StringArray.msg
)

add_service_files(
  FILES
  claim_track.srv
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS rospy roscpp rospy std_msgs message_runtime
  DEPENDS roscpp rospy
  gazebo_ros
)

## Your package locations should be listed before other locations
include_directories(
include/${PROJECT_NAME} ${Boost_INCLUDE_DIR}
  ${catkin_INCLUDE_DIRS} ${GAZEBO_INCLUDE_DIRS}
)

file(GLOB python_scripts_DIR
    "scripts/*.py"
)

catkin_install_python(PROGRAMS ${python_scripts_DIR} DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
install(DIRECTORY launch DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
add_executable(onion_blocks_spawner src/onion_blocks_spawner.cpp)
add_dependencies(onion_blocks_spawner sawyer_irl_project_generate_messages_cpp)
add_executable(onion_blocks_poses_publisher src/onion_blocks_poses_publisher.cpp)
add_dependencies(onion_blocks_poses_publisher sawyer_irl_project_generate_messages_cpp)
add_executable(planning_scene_sawyer src/planning_scene_sawyer.cpp)
add_dependencies(robotiq_2f_gripper_control_generate_messages_cpp ${catkin_EXPORTED_TARGETS})
target_link_libraries(planning_scene_sawyer ${catkin_LIBRARIES} ${Boost_LIBRARIES})
target_link_libraries(onion_blocks_spawner ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GAZEBO_LIBRARIES})
target_link_libraries(onion_blocks_poses_publisher ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${GAZEBO_LIBRARIES})
install(TARGETS planning_scene_sawyer DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})