cmake_minimum_required(VERSION 3.0.2)
project(ur_client_library)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules/" ${CMAKE_MODULE_PATH})

if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  message("${PROJECT_NAME}: You did not request a specific build type: selecting 'RelWithDebInfo'.")
  set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

option(WITH_ASAN "Compile with address sanitizer support" OFF)

##
## Check C++11 support / enable global pedantic and Wall
##
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic")

add_library(urcl STATIC
    src/comm/tcp_socket.cpp
    src/comm/tcp_server.cpp
    src/control/reverse_interface.cpp
    src/control/script_sender.cpp
    src/control/trajectory_point_interface.cpp
    src/control/script_command_interface.cpp
    src/primary/primary_package.cpp
    src/primary/robot_message.cpp
    src/primary/robot_state.cpp
    src/primary/robot_message/version_message.cpp
    src/primary/robot_state/kinematics_info.cpp
    src/rtde/control_package_pause.cpp
    src/rtde/control_package_setup_inputs.cpp
    src/rtde/control_package_setup_outputs.cpp
    src/rtde/control_package_start.cpp
    src/rtde/data_package.cpp
    src/rtde/get_urcontrol_version.cpp
    src/rtde/request_protocol_version.cpp
    src/rtde/rtde_package.cpp
    src/rtde/text_message.cpp
    src/rtde/rtde_client.cpp
    src/ur/ur_driver.cpp
    src/ur/calibration_checker.cpp
    src/ur/dashboard_client.cpp
    src/ur/tool_communication.cpp
    src/ur/robot_receive_timeout.cpp
    src/ur/version_information.cpp
    src/rtde/rtde_writer.cpp
    src/default_log_handler.cpp
    src/log.cpp
    src/helpers.cpp
)
add_library(ur_client_library::urcl ALIAS urcl)
target_compile_options(urcl PRIVATE -Wall -Wextra -Wno-unused-parameter)
target_link_options(
   urcl PUBLIC $<BUILD_INTERFACE:$<$<CXX_COMPILER_ID:GNU>:-static-libgcc;-static-libstdc++>>
)
if(WITH_ASAN)
  target_compile_options(urcl PUBLIC -fsanitize=address)
  target_link_options(urcl PUBLIC -fsanitize=address)
endif()
target_include_directories( urcl PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)
set_target_properties(urcl PROPERTIES POSITION_INDEPENDENT_CODE ON)

find_package(Threads REQUIRED)
if(THREADS_HAVE_PTHREAD_ARG)
  target_compile_options(urcl PUBLIC "-pthread")
endif()
if(CMAKE_THREAD_LIBS_INIT)
  target_link_libraries(urcl PUBLIC "${CMAKE_THREAD_LIBS_INIT}")
endif()

##
## Build testing if enabled by option
##
if (BUILDING_TESTS)
  enable_testing()
  add_subdirectory(tests)
else()
  message(STATUS "Building tests disabled.")
endif()
