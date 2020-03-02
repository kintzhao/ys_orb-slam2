cmake_minimum_required(VERSION 2.8.3)
project(ORB_SLAM2)

IF(NOT ROS_BUILD_TYPE)
  SET(ROS_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${ROS_BUILD_TYPE})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11") 

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3 -march=native")

set(GLOG_LIBRARY glog)

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/../../../cmake_modules)

find_package(catkin REQUIRED COMPONENTS roscpp cv_bridge image_transport tf)

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

find_package(PCL REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})


set(ORB_SOURCE_DIR  "${PROJECT_SOURCE_DIR}/../../../")
include_directories(
${ORB_SOURCE_DIR}
${ORB_SOURCE_DIR}/include
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
${catkin_INCLUDE_DIRS}
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/src
)

catkin_package( CATKIN_DEPENDS  roscpp  )

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${ORB_SOURCE_DIR}/lib)
add_library(ORB_SLAM2 SHARED
${ORB_SOURCE_DIR}/src/System.cc
${ORB_SOURCE_DIR}/src/Tracking.cc
${ORB_SOURCE_DIR}/src/LocalMapping.cc
${ORB_SOURCE_DIR}/src/LoopClosing.cc
${ORB_SOURCE_DIR}/src/ORBextractor.cc
${ORB_SOURCE_DIR}/src/ORBmatcher.cc
${ORB_SOURCE_DIR}/src/FrameDrawer.cc
${ORB_SOURCE_DIR}/src/Converter.cc
${ORB_SOURCE_DIR}/src/MapPoint.cc
${ORB_SOURCE_DIR}/src/KeyFrame.cc
${ORB_SOURCE_DIR}/src/Map.cc
${ORB_SOURCE_DIR}/src/MapDrawer.cc
${ORB_SOURCE_DIR}/src/Optimizer.cc
${ORB_SOURCE_DIR}/src/PnPsolver.cc
${ORB_SOURCE_DIR}/src/Frame.cc
${ORB_SOURCE_DIR}/src/KeyFrameDatabase.cc
${ORB_SOURCE_DIR}/src/Sim3Solver.cc
${ORB_SOURCE_DIR}/src/Initializer.cc
${ORB_SOURCE_DIR}/src/Viewer.cc
src/MapPublisher.cc
)

target_link_libraries(ORB_SLAM2
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${catkin_LIBRARIES}
${PCL_LIBRARIES}
${ORB_SOURCE_DIR}/Thirdparty/DBoW2/lib/libDBoW2.so
${ORB_SOURCE_DIR}/Thirdparty/g2o/lib/libg2o.so
${GLOG_LIBRARY}
)


set(LIBS 
${catkin_LIBRARIES}
${OpenCV_LIBS} 
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/../../../Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/../../../Thirdparty/g2o/lib/libg2o.so
#${PROJECT_SOURCE_DIR}/../../../lib/libORB_SLAM2.so
${GLOG_LIBRARY}
ORB_SLAM2
)


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

add_executable(RGBD_rviz src/ros_rgbd_rviz.cc src/MapPublisher.cc)
target_link_libraries(RGBD_rviz ${LIBS})


## Install executables and/or libraries
install(TARGETS mono MonoAR Stereo RGBD
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
 

install(DIRECTORY launch  conf
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)



