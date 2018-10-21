cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project(SOID)

file(GLOB SOURCE
		"./src/*.c"
	)
include_directories("./include")
add_library(soid ${SOURCE})