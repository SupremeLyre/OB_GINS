cmake_minimum_required(VERSION 3.10)
project(ob_gins)

set(CMAKE_CXX_STANDARD 20)

if(CMAKE_COMPILER_IS_GNUCXX)
    # 检查GNU是否支持-Wa,-mbig-obj
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-Wa,-mbig-obj" GNU_BIG_OBJ_FLAG_ENABLE)
    message(STATUS GNU_BIG_OBJ_FLAG_ENABLE=${GNU_BIG_OBJ_FLAG_ENABLE})
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
target_compile_options(${PROJECT_NAME} PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/bigobj>
    $<$<AND:$<CXX_COMPILER_ID:GNU>,$<BOOL:${GNU_BIG_OBJ_FLAG_ENABLE}>>:-Wa,-mbig-obj>)

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
if(CMAKE_SYSTEM_NAME MATCHES "Windows")
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${PROJECT_SOURCE_DIR}/src/thirdparty/Ceres/bin/libglog.dll
        ${EXECUTABLE_OUTPUT_PATH}/libglog.dll
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${PROJECT_SOURCE_DIR}/src/thirdparty/Ceres/bin/libceres.dll
        ${EXECUTABLE_OUTPUT_PATH}/libceres.dll
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${PROJECT_SOURCE_DIR}/src/thirdparty/Ceres/bin/libgflags.dll
        ${EXECUTABLE_OUTPUT_PATH}/libgflags.dll
    # 如有其它依赖 DLL，继续添加
)
endif()