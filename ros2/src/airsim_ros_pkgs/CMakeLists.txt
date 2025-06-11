cmake_minimum_required(VERSION 3.5)
project(airsim_ros_pkgs)

find_package(rclcpp REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(geographic_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_sensor_msgs REQUIRED)
find_package(rclpy REQUIRED)
find_package(tf2 REQUIRED)
find_package(image_transport REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(tf2_eigen REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(mavros_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(ament_cmake REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rosidl_typesupport_cpp REQUIRED)
find_package(rosidl_default_runtime REQUIRED)
find_package(airsim_interfaces REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)

set(AIRSIM_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../../../)

add_subdirectory("${AIRSIM_ROOT}/cmake/rpclib_wrapper" rpclib_wrapper)
add_subdirectory("${AIRSIM_ROOT}/cmake/AirLib" AirLib)
add_subdirectory("${AIRSIM_ROOT}/cmake/MavLinkCom" MavLinkCom)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_FLAGS "-O3 -Wall -Wextra -Wnoexcept -Wstrict-null-sentinel")

set(RPC_LIB_INCLUDES "${AIRSIM_ROOT}/external/rpclib/rpclib-2.3.0/include")
set(RPC_LIB rpc)
message(STATUS "found RPC_LIB_INCLUDES=${RPC_LIB_INCLUDES}")

set(INCLUDE_DIRS
  include
  ${AIRSIM_ROOT}/AirLib/include
  ${RPC_LIB_INCLUDES}
  ${AIRSIM_ROOT}/MavLinkCom/include
  ${AIRSIM_ROOT}/MavLinkCom/common_utils
  ${EIGEN3_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS}
  $<TARGET_PROPERTY:tf2::tf2,INTERFACE_INCLUDE_DIRECTORIES>
  $<TARGET_PROPERTY:tf2_geometry_msgs::tf2_geometry_msgs,INTERFACE_INCLUDE_DIRECTORIES>
)

include_directories(
	${INCLUDE_DIRS}
)
add_definitions(-DEIGEN_NO_DEPRECATED_WARNING)

add_library(airsim_settings_parser src/airsim_settings_parser.cpp)
target_link_libraries(airsim_settings_parser AirLib)

add_library(pd_position_controller_simple src/pd_position_controller_simple.cpp)
target_link_libraries(
  pd_position_controller_simple
  AirLib
  tf2::tf2
)
ament_target_dependencies(pd_position_controller_simple 
  rclcpp
  nav_msgs
  geometry_msgs
  tf2
  tf2_eigen
  airsim_interfaces
)
add_library(airsim_ros src/airsim_ros_wrapper.cpp)
target_link_libraries(airsim_ros ${OpenCV_LIBS} yaml-cpp AirLib airsim_settings_parser)
ament_target_dependencies(airsim_ros 
  rclcpp
  sensor_msgs
  geometry_msgs
  nav_msgs
  image_transport
  tf2_ros
  tf2_eigen
  cv_bridge
  airsim_interfaces
  std_srvs
  tf2
  tf2_sensor_msgs
)

add_executable(airsim_node src/airsim_node.cpp)
target_link_libraries(airsim_node airsim_ros AirLib)
ament_target_dependencies(airsim_node rclcpp)

add_executable(pd_position_controller_simple_node src/pd_position_controller_simple_node.cpp)
target_link_libraries(
  pd_position_controller_simple_node
  pd_position_controller_simple
  airsim_ros
  AirLib
  tf2::tf2
)

ament_target_dependencies(pd_position_controller_simple_node
  rclcpp
  tf2
)

install(TARGETS
  airsim_node
  pd_position_controller_simple_node
  DESTINATION lib/${PROJECT_NAME}
)

install(TARGETS
  airsim_ros
  pd_position_controller_simple
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)

install(FILES README.md DESTINATION share/${PROJECT_NAME})
install(DIRECTORY launch rviz DESTINATION share/${PROJECT_NAME})

ament_export_dependencies(
  rclcpp nav_msgs geographic_msgs std_srvs tf2_ros tf2_sensor_msgs rclpy
  tf2 image_transport geometry_msgs cv_bridge tf2_geometry_msgs tf2_eigen
  sensor_msgs mavros_msgs rosidl_default_generators ament_cmake std_msgs
  rosidl_default_runtime airsim_interfaces
)

ament_export_include_directories(${INCLUDE_DIRS})
ament_export_libraries(
  airsim_settings_parser
  pd_position_controller_simple
  airsim_ros
)

ament_package()

