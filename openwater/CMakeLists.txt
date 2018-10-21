cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project(OpenWater)

file(GLOB SOURCE
		"./src/*.cpp"
		"./learnopengl/*.h"
		"./*.h"
	)

#message(STATUS ${SOURCE})

add_executable(OpenWater ${SOURCE})