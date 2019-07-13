# 指定 cmake版本
# 指定 project name
# 指定生成文件的输出目录
cmake_minimum_required(VERSION 2.8)
project(project_name)

message("-- system arch:  ${CMAKE_SYSTEM_PROCESSOR}")
message("-- output path:  ${PROJECT_OUTPUT_DIR} ")

# set output path
set(PROJECT_OUTPUT_DIR ${PROJECT_BINARY_DIR}/${CMAKE_SYSTEM_PROCESSOR})
SET(PROJECT_INCLUDE_DIR ${PROJECT_OUTPUT_DIR}/include)

# mkdir directory
file(MAKE_DIRECTORY ${PROJECT_INCLUDE_DIR}) 
FILE(MAKE_DIRECTORY ${PROJECT_OUTPUT_DIR}/bin)
FILE(MAKE_DIRECTORY ${PROJECT_OUTPUT_DIR}/lib)

## set the target path
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_OUTPUT_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_OUTPUT_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_OUTPUT_DIR}/lib)

# 中间部分-指定工程依赖
# 指定工程需要用到的一些系统包和库
# 指定依赖的库文件

find_package(PkgConfig)
	pkg_check_modules(GSTREAMER QUIET gstreamer-1.0)
	pkg_check_modules(GST_APP REQUIRED gstreamer-app-1.0)

find_package(Qt4 REQUIRED)
find_package(CUDA)
find_package(OpenGL)
find_package(OpenCV)

## set c++11 编译选项
# -std=c++11 包含C++11特性
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")	

## set CUDA NVCC编译选项
set(
	CUDA_NVCC_FLAGS
	${CUDA_NVCC_FLAGS}; 
    -O3 
	-gencode arch=compute_53,code=sm_53
    -gencode arch=compute_62,code=sm_62  ##TX2
    -gencode arch=compute_30,code=sm_30
)

# INCLUDE for c/C++ include
include_directories(${PROJECT_INCLUDE_DIR})
include_directories(
    /usr/include/gstreamer-1.0
    /usr/lib/${CMAKE_SYSTEM_PROCESSOR}-linux-gnu/gstreamer-1.0/include
    /usr/include/glib-2.0
    /usr/include/libxml2 /usr/lib/${CMAKE_SYSTEM_PROCESSOR}-linux-gnu/glib-2.0/include
    /usr/include/opencv2
)

# 结尾部分-指定输出文件

#create executable-file
add_executable(exe-filename needed-C_or_C++file)

#create shared-library
add_library(lib-filename SHARED needed-files-list)

## CUDA 版本
cuda_add_executable(exe-filename needed-cu_file)
cuda_add_library(lib-filename SHARED needed-cu_files-list)

## 对于有多个文件可以先创建一个变量 file-list
file(GLOB gstcam_source  lib/gstreamer/*cpp  lib/image/*cpp lib/cuda/*cpp lib/cuda/*cu  lib/opengl/*cpp  )
file(GLOB gstcam_include lib/gstreamer/*h  lib/image/*h lib/cuda/*h lib/opengl/*h)

## 循环打印一下看看
foreach(sources ${gstcam_source})
MESSAGE("-- the ${PROJECT_NAME}  contion ${sources}")     
endforeach(sources ${})


## creat gstcam shared library
cuda_add_library(${PROJECT_NAME} SHARED ${gstcam_source})
target_link_libraries(${PROJECT_NAME}    Qt4::QtGui GL GLEW gstreamer-1.0 gstapp-1.0 ${OpenCV_LIBS})

# transfer all headers to the include directory
foreach(include ${gstcam_include})
   message("-- Copying ${include}")
   configure_file(${include} ${PROJECT_INCLUDE_DIR} COPYONLY)
endforeach()

## create exe-file
add_executable(gst_app_test src/gst-camera.cpp)
target_link_libraries(gst_app_test ${PROJECT_NAME})

