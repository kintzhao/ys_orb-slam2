cmake_minimum_required(VERSION 2.8.3)
project(ORB_SLAM2)

IF(NOT ROS_BUILD_TYPE)
  SET(ROS_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${ROS_BUILD_TYPE})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11") 

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../../../cmake_modules)

find_package(catkin REQUIRED COMPONENTS roscpp cv_bridge image_transport)

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)


include_directories(
${catkin_INCLUDE_DIRS}
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/../../../
${PROJECT_SOURCE_DIR}/../../../include
${Pangolin_INCLUDE_DIRS}
)

set(LIBS 
${catkin_LIBRARIES}
${OpenCV_LIBS} 
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/../../../Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/../../../Thirdparty/g2o/lib/libg2o.so
${PROJECT_SOURCE_DIR}/../../../lib/libORB_SLAM2.so
)

catkin_package( CATKIN_DEPENDS  roscpp  )

add_executable(mono         src/ros_mono.cc)
target_link_libraries(mono  ${LIBS} )

# Node for monocular camera (Augmented Reality Demo)
add_executable(MonoAR src/AR/ros_mono_ar.cc src/AR/ViewerAR.h src/AR/ViewerAR.cc )
target_link_libraries(MonoAR ${LIBS} )

# Node for stereo camera
add_executable(Stereo src/ros_stereo.cc )
target_link_libraries(Stereo ${LIBS} )

# Node for RGB-D camera
add_executable(RGBD src/ros_rgbd.cc )
target_link_libraries(RGBD ${LIBS} )


## Install executables and/or libraries
install(TARGETS mono MonoAR Stereo RGBD
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
 

install(DIRECTORY launch  conf
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)



