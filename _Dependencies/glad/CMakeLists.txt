cmake_minimum_required (VERSION 3.6)
project(libglad_static)

add_library(libglad_static STATIC ${CMAKE_SOURCE_DIR}/src/glad.c)
target_include_directories(libglad_static PUBLIC ${CMAKE_SOURCE_DIR}/include)
