cmake_minimum_required (VERSION 3.6)
project(libglad_static)

add_library(libglad_static STATIC ${PROJECT_SOURCE_DIR}/src/glad.c)
target_include_directories(libglad_static PUBLIC ${PROJECT_SOURCE_DIR}/include)
