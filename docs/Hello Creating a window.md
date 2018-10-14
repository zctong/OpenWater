# Hello Creating a window

## 工程参考
- zctong的工程：https://github.com/zctong/OpenWater.git

## 工具准备

### cmake大法
- cmake：https://cmake.org/
    - 后面使用介绍

### IDE
- vs2017 or xcode

## 工程结构
- openwater：每节作业源码
- external：第三方库
    - glad：opengl接口管理库
    - glfw：opengl窗口管理库
- licenses：开源证书
- docs：学习笔记

## gl各种库概念
- OpenGL函数库相关的API有核心库(gl)，实用库(glu)，辅助库(aux)、实用工具库(glut)，窗口库(glx、agl、wgl)和扩展函数库等

- 窗口api交互
    - EGL:官方出现，opengl3标准中的内容。EGL is an interface between Khronos rendering APIs (such as OpenGL, OpenGL ES or OpenVG) and the underlying native platform windowing system. EGL handles graphics context management, surface/buffer binding, rendering synchronization, and enables "high-performance, accelerated, mixed-mode 2D and 3D rendering using other Khronos APIs 
    
    - glut：比较旧的窗口api封装，现在不流行。GLUT is the OpenGL Utility Toolkit, a window system independent toolkit for writing OpenGL programs. It implements a simple windowing application programming interface (API) for OpenGL. GLUT makes it considerably easier to learn about and explore OpenGL programming. GLUT provides a portable API so you can write a single OpenGL program that works across all PC and workstation OS platforms.
    
    - glfw：比较流行这个。GLFW is an Open Source, multi-platform library for OpenGL, OpenGL ES and Vulkan development on the desktop. It provides a simple API for creating windows, contexts and surfaces, receiving input and events.
    
    - cocos引擎使用的glfw，本课程也是使用glfw

- 各种opengl api封装
    - glew:The OpenGL Extension Wrangler Library (GLEW) is a cross-platform open-source C/C++ extension loading library. GLEW provides efficient run-time mechanisms for determining which OpenGL extensions are supported on the target platform. OpenGL core and extension functionality is exposed in a single header file. GLEW has been tested on a variety of operating systems, including Windows, Linux, Mac OS X, FreeBSD, Irix, and Solaris.
    
    - glad:GL/GLES/EGL/GLX/WGL Loader-Generator based on the official specs.

    - cocos引擎使用的是glew，本课程新版改成了使用glad

- glad vs glew
    - [对比讨论](https://www.reddit.com/r/opengl/comments/3m28x1/glew_vs_glad/)


## cmake构建说明
        
    cmake_minimum_required (VERSION 2.8)
    cmake_policy(VERSION 2.8)
    
    project (OpenWater)
    
    list(APPEND CMAKE_CXX_FLAGS "-std=c++11")
    
    
    # debug/release 环境设置
    SET(CMAKE_DEBUG_POSTFIX "_d" CACHE STRING "add a postfix, usually d on windows")  
    SET(CMAKE_RELEASE_POSTFIX "" CACHE STRING "add a postfix, usually empty on windows")  
    
    SET(LIBRARY_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/libs)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)  
    
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
    
    set(LIBS ${GLFW3} ${GLAD} opengl32)
    message(STATUS, ${LIBS})
    
    
    set(CHAPTERS
        1.getting_started
    )
    
    set(1.getting_started
        1.1.hello_window
    )
    
    # then create a project file per tutorial
    foreach(CHAPTER ${CHAPTERS})
        foreach(DEMO ${${CHAPTER}})
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
    		add_dependencies(${NAME} glfw)		#添加工程编译依赖，不知道为啥glad默认关联了
            
    		# copy shader files to build directory
            add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E make_directory $<TARGET_FILE_DIR:${NAME}>)
    		set(RESBASEPATH $<TARGET_FILE_DIR:${NAME}>)
    		add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E make_directory "${RESBASEPATH}/resources")
            foreach(SHADER ${SHADERS})
    			add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${NAME}>)
    		endforeach(SHADER)
    		add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/resources" "${RESBASEPATH}/resources")
    		
        endforeach(DEMO)
    endforeach(CHAPTER)

