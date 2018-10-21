cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project(test C)

#turn on using solution folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(CHAPTERS
    1.getting_started
)

set(1.getting_started
    1.1.hello_window
    1.2.hello_window_clear
    2.1.hello_triangle
    2.2.hello_triangle_indexed
    2.3.hello_triangle_exercise1
    2.4.hello_triangle_exercise2
    2.5.hello_triangle_exercise3
    3.1.shaders_uniform
    3.2.shaders_interpolation
    3.3.shaders_class
    4.1.textures
    4.2.textures_combined
    4.3.textures_exercise2
    4.4.textures_exercise3
    4.5.textures_exercise4
)

# then create a project file per tutorial

foreach(CHAPTER ${CHAPTERS})
	set_property(TARGET PROPERTY FOLDER ${CHAPTER})
    foreach(DEMO ${${CHAPTER}})
		file(GLOB SHADERS
                 "${CHAPTER}/${DEMO}/*.vs"
                 "${CHAPTER}/${DEMO}/*.fs"
                 "${CHAPTER}/${DEMO}/*.gs"
        )
		file(GLOB DLLFILE 
				"libs/*.dll"
		)
		file(GLOB ALLFILE
			"${CHAPTER}/${DEMO}/*.h"
            "${CHAPTER}/${DEMO}/*.cpp"
			"${CHAPTER}/${DEMO}/*.vs"
			 "${CHAPTER}/${DEMO}/*.fs"
			 "${CHAPTER}/${DEMO}/*.gs"
		)

        set(NAME "${DEMO}")
		#message(STATUS, ${NAME}, "工程名字")
		#message(STATUS, ${SHADERS}, "shader文件")
        add_executable(${NAME} ${ALLFILE})
        target_link_libraries(${NAME} ${LIBS})
		add_dependencies(${NAME} glfw)		#添加工程编译依赖，不知道为啥glad默认关联了
        
		set_property(TARGET ${NAME} PROPERTY FOLDER "Test-Projects")
		
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