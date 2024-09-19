cmake_minimum_required(VERSION 3.8)
project(bumperbot_firmware)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_python REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclpy REQUIRED)
find_package(std_msgs REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)

# Find the LibSerial package
find_package(PkgConfig REQUIRED)
pkg_check_modules(SERIAL REQUIRED libserial)

ament_python_install_package(${PROJECT_NAME})

# Create the shared library for bumperbot firmware
add_library(${PROJECT_NAME} SHARED 
  src/bumperbot_interface.cpp
)

# Include directories for the bumperbot firmware
target_include_directories(${PROJECT_NAME} PRIVATE 
  include
  ${SERIAL_INCLUDE_DIRS}
)

# Link the library with the LibSerial library
target_link_libraries(${PROJECT_NAME} 
  ${SERIAL_LIBRARIES}
)

# Add dependencies for ament and ROS packages
ament_target_dependencies(${PROJECT_NAME}
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
)

# Export plugin for the hardware interface
pluginlib_export_plugin_description_file(hardware_interface bumperbot_interface.xml)

# Create executable for simple_serial_receiver
add_executable(simple_serial_receiver src/simple_serial_receiver.cpp)
ament_target_dependencies(simple_serial_receiver rclcpp std_msgs)

# Include directories and link libraries for simple_serial_receiver
target_include_directories(simple_serial_receiver PUBLIC
  ${SERIAL_INCLUDE_DIRS}
)
target_link_libraries(simple_serial_receiver
  ${SERIAL_LIBRARIES}
)

# Create executable for simple_serial_transmitter
add_executable(simple_serial_transmitter src/simple_serial_transmitter.cpp)
ament_target_dependencies(simple_serial_transmitter rclcpp std_msgs)

# Include directories and link libraries for simple_serial_transmitter
target_include_directories(simple_serial_transmitter PUBLIC
  ${SERIAL_INCLUDE_DIRS}
)
target_link_libraries(simple_serial_transmitter
  ${SERIAL_LIBRARIES}
)

# Install the bumperbot firmware library
install(
  TARGETS ${PROJECT_NAME}
  DESTINATION lib
)

# Install the include directory
install(
  DIRECTORY include
  DESTINATION include
)

# Install the launch directory
install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)

# Install Python scripts
install(PROGRAMS
  ${PROJECT_NAME}/simple_serial_transmitter.py
  ${PROJECT_NAME}/simple_serial_receiver.py
  ${PROJECT_NAME}/mpu6050_driver.py
  DESTINATION lib/${PROJECT_NAME}
)

# Install the executables
install(TARGETS
  simple_serial_receiver
  simple_serial_transmitter
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  DESTINATION lib/${PROJECT_NAME}
)

# Export include directories and libraries for ament
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

# Export dependencies for ament
ament_export_dependencies(
  hardware_interface
  pluginlib
  rclcpp
  rclcpp_lifecycle
)

ament_package()