cmake_minimum_required(VERSION 2.8.3)
project(opencv2_catkin)

find_package(catkin_simple REQUIRED)

catkin_simple()

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)

ExternalProject_Add(opencv2_src
  GIT_REPOSITORY https://github.com/opencv/opencv.git
  GIT_TAG 82f8176b0634c5d744d1a45246244291d895b2d1
  UPDATE_COMMAND ""
  CONFIGURE_COMMAND cd ../opencv2_src && mkdir build && cd build && cmake 
    -DWITH_TBB=ON
    -DBUILD_NEW_PYTHON_SUPPORT=ON 
    -DWITH_V4L=ON 
    -DINSTALL_C_EXAMPLES=OFF
    -DINSTALL_PYTHON_EXAMPLES=OFF 
    -DBUILD_EXAMPLES=OFF
    -DWITH_QT=OFF # Needed by clang under Ubuntu 14.04 and GTK_WIDGET(cvGetWindowHandle(...)) with gcc (image_view) 
    -DWITH_OPENGL=ON 
    -DWITH_VTK=ON
    -DENABLE_PRECOMPILED_HEADERS=OFF
    -DCMAKE_INSTALL_PREFIX=${CATKIN_DEVEL_PREFIX} 
    -DBUILD_SHARED_LIBS=ON 
    -DWITH_CUDA=OFF 
    -DWITH_OPENCL=OFF
    -DBUILD_opencv_gpu=OFF 
    -DBUILD_opencv_ts=OFF 
    -DBUILD_TESTS=OFF -DBUILD_opencv_ts=OFF ..
  BUILD_COMMAND cd ../opencv2_src/build && make -j8
  INSTALL_COMMAND cd ../opencv2_src/build && make install -j8
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cc)
add_dependencies(${PROJECT_NAME} opencv2_src)
target_link_libraries(${PROJECT_NAME} 
  ${CATKIN_DEVEL_PREFIX}/lib/libopencv_core${CMAKE_SHARED_LIBRARY_SUFFIX})

cs_install()
install(DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/opencv2
  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.hpp"
  PATTERN "*.h"
  PATTERN ".svn" EXCLUDE
)

cs_export(
  INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS opencv2-extras.cmake)
