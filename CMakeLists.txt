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

