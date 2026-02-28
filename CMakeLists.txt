cmake_minimum_required(VERSION 3.10)
project(ob_gins)

set(CMAKE_CXX_STANDARD 20)

if(MSVC)
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /STACK:8388608")
endif()

set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

file(GLOB_RECURSE SOURCE
        src/ob_gins.cc
        src/common/*.cc
        src/fileio/*.cc
        src/factors/*.cc
        src/preintegration/*.cc
)

include_directories(${PROJECT_SOURCE_DIR})
add_executable(${PROJECT_NAME} ${SOURCE})
if(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _USE_MATH_DEFINES)
endif()
# Eigen3
set(Eigen3_DIR ${PROJECT_SOURCE_DIR}/src/thirdparty/Ceres/share/eigen3/cmake)
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})

# yaml-cpp
set(YAML_CPP_BUILD_TOOLS OFF)
add_subdirectory(src/thirdparty/yaml-cpp)
target_link_libraries(${PROJECT_NAME} yaml-cpp)

# abseil
set(ABSL_PROPAGATE_CXX_STD true)
add_subdirectory(src/thirdparty/abseil-cpp)
target_link_libraries(${PROJECT_NAME}
        absl::strings
        absl::str_format
        absl::time)

# Ceres
if(CMAKE_SYSTEM_NAME MATCHES "Windows")
    set(Ceres_DIR ${PROJECT_SOURCE_DIR}/src/thirdparty/Ceres/lib/cmake/Ceres)
    set(glog_DIR ${PROJECT_SOURCE_DIR}/src/thirdparty/Ceres/lib/cmake/glog)
endif()
find_package(Ceres REQUIRED)
target_link_libraries(${PROJECT_NAME} Ceres::ceres)