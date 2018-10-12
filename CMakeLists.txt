cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project (LearnOpenGL)

# debug/release 环境设置
SET(CMAKE_DEBUG_POSTFIX "_d" CACHE STRING "add a postfix, usually d on windows")  
SET(CMAKE_RELEASE_POSTFIX "" CACHE STRING "add a postfix, usually empty on windows")  

# Set the build postfix extension according to what configuration is being built.  
IF (CMAKE_BUILD_TYPE MATCHES "Release")  
    SET(CMAKE_BUILD_POSTFIX "${CMAKE_RELEASE_POSTFIX}")  
ELSEIF(CMAKE_BUILD_TYPE MATCHES "Debug")  
    SET(CMAKE_BUILD_POSTFIX "${CMAKE_DEBUG_POSTFIX}")  
ELSE()  
    SET(CMAKE_BUILD_POSTFIX "")  
ENDIF()  

message(STATUS, ${CMAKE_DEBUG_POSTFIX}, "debug 后缀")
message(STATUS, ${CMAKE_RELEASE_POSTFIX}, "release 后缀")
message(STATUS, ${CMAKE_SOURCE_DIR}, "根路径")

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

link_directories(${CMAKE_SOURCE_DIR}/libs)
include_directories(${CMAKE_SOURCE_DIR}/includes)
#include_directories(${CMAKE_SOURCE_DIR}/breakpad)

list(APPEND CMAKE_CXX_FLAGS "-std=c++11")


set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules/")
#find the required packages
#find_package(GLM REQUIRED)
#message(STATUS "GLM included at ${GLM_INCLUDE_DIR}")
find_package(GLFW3 REQUIRED)
message(STATUS "Found GLFW3 in ${GLFW3_INCLUDE_DIR}")
message(STATUS "Found GLFW3 in ${GLFW3_LIBRARY}")
#find_package(ASSIMP REQUIRED)
#message(STATUS "Found ASSIMP in ${ASSIMP_INCLUDE_DIR}")

set(GLFW3 "glfw3${CMAKE_DEBUG_POSTFIX}")
#set(ASSIMPLIB "assimp-vc140-mt${CMAKE_DEBUG_POSTFIX}")

set(LIBS GLFW3 opengl32)

set(CHAPTERS
    1.getting_started
)

set(1.getting_started
    1.1.hello_window
)

configure_file(config/root_directory.h.in config/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/config)


# first create relevant static libraries requried for other projects
#add_library(STB_IMAGE "openwater/stb_image.cpp")
#set(LIBS ${LIBS} STB_IMAGE)

add_library(GLAD "openwater/glad.c")
set(LIBS ${LIBS} GLAD)



# then create a project file per tutorial
foreach(CHAPTER ${CHAPTERS})
    foreach(DEMO ${${CHAPTER}})
        file(GLOB SOURCE
            "openwater/${CHAPTER}/${DEMO}/*.h"
            "openwater/${CHAPTER}/${DEMO}/*.cpp"
        )
		file(GLOB SHADERS
                 "openwater/${CHAPTER}/${DEMO}/*.vs"
                 "openwater/${CHAPTER}/${DEMO}/*.fs"
                 "openwater/${CHAPTER}/${DEMO}/*.gs"
        )
		file(GLOB DLLFILE 
				"libs/*.dll"
		)
		file(GLOB ALLFILE
			"openwater/${CHAPTER}/${DEMO}/*.h"
            "openwater/${CHAPTER}/${DEMO}/*.cpp"
			"openwater/${CHAPTER}/${DEMO}/*.vs"
			 "openwater/${CHAPTER}/${DEMO}/*.fs"
			 "openwater/${CHAPTER}/${DEMO}/*.gs"
		)

        set(NAME "${DEMO}")
		message(STATUS, ${NAME}, "工程名字")
		message(STATUS, ${ALLFILE}, "所有文件")
        add_executable(${NAME} ${ALLFILE})
        target_link_libraries(${NAME} ${LIBS})
        set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/")
        # copy shader files to build directory
		#message(STATUS "--------------${NAME}")
		#message(STATUS ${ALLFILE})
        add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${NAME}>)
		set(RESBASEPATH $<TARGET_FILE_DIR:${NAME}>)
		add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E make_directory "${RESBASEPATH}/resources")
        foreach(SHADER ${SHADERS})
			# configure_file(${SHADER} "test")
			#add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E remove -f $<TARGET_FILE_DIR:${NAME}>)
			add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${NAME}>)
		endforeach(SHADER)
		foreach(DLL ${DLLFILE})
			add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${DLL} $<TARGET_FILE_DIR:${NAME}>)
        endforeach(DLL)
		add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/resources" "${RESBASEPATH}/resources")
		#message(STATUS $<TARGET_FILE_DIR:${NAME}>"--------------------------")
        # if compiling for visual studio, also use configure file for each project (specifically to set up working directory)
        #if(MSVC)
        #  configure_file(${CMAKE_SOURCE_DIR}/configuration/visualstudio.vcxproj.user.in #${CMAKE_CURRENT_BINARY_DIR}/${NAME}.vcxproj.user @ONLY)
        #endif(MSVC)
    endforeach(DEMO)
endforeach(CHAPTER)

