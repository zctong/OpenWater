cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project (OpenWater)

list(APPEND CMAKE_CXX_FLAGS "-std=c++11")


# debug/release 环境设置
SET(CMAKE_DEBUG_POSTFIX "_d" CACHE STRING "add a postfix, usually d on windows")  
SET(CMAKE_RELEASE_POSTFIX "" CACHE STRING "add a postfix, usually empty on windows")  

SET(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/libs)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)  

#将配置自动生成的头文件生成到头文件搜索路径中目录中
configure_file(config/root_directory.h.in ${CMAKE_SOURCE_DIR}/openwater/root_directory.h)

link_directories(${LIBRARY_OUTPUT_PATH})

# first create relevant static libraries requried for other projects
# add glfw lib
set(GLFW_SOURCE "${CMAKE_SOURCE_DIR}/external/glfw/")
add_subdirectory(${GLFW_SOURCE} glfw3)
include_directories(${GLFW_SOURCE}/include)
# link different lib according to different build type
SET(GLFW3 optimized glfw3 debug "glfw3${CMAKE_DEBUG_POSTFIX}")

# add glad lib
set(GLAD_SOURCE "${CMAKE_SOURCE_DIR}/external/glad/")
add_subdirectory(${GLAD_SOURCE} glad)
include_directories(${GLAD_SOURCE}/include)
SET(GLAD optimized glad debug "glad${CMAKE_DEBUG_POSTFIX}")

# add soil lib
#set(SOIL_SOURCE "${CMAKE_SOURCE_DIR}/external/soil/")
#add_subdirectory(${SOIL_SOURCE} soil)
#include_directories(${SOIL_SOURCE}/include)
#SET(SOID optimized soid debug "soid${CMAKE_DEBUG_POSTFIX}")

# add glm header file
set(GLM_SOURCE "${CMAKE_SOURCE_DIR}/external/glm/")
#add_subdirectory(${GLM_SOURCE} glm)
include_directories(${GLM_SOURCE})
#SET(GLM optimized glm debug "glm${CMAKE_DEBUG_POSTFIX}")


set(LIBS ${GLFW3} ${GLAD} ${GLM} opengl32)
message(STATUS, ${LIBS})

# 添加引擎project
include_directories(${CMAKE_SOURCE_DIR}/openwater)
add_subdirectory(${CMAKE_SOURCE_DIR}/openwater OpenWater)

# 添加测试工程
add_subdirectory(${CMAKE_SOURCE_DIR}/test)

