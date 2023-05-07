#-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Author: Inan Evin
# www.inanevin.com
# 
# Copyright (C) 2022 Inan Evin
# 
# Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, 
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions 
# and limitations under the License.
#-------------------------------------------------------------------------------------------------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.6)
project(Examples)

#--------------------------------------------------------------------
# Set sources
#--------------------------------------------------------------------

set(EXAMPLES_SOURCES

        src/Main.cpp
        src/DemoScreens.cpp
        src/Backends/OpenGLGLFW.cpp
        )

set(EXAMPLES_HEADERS

        include/Main.hpp
        include/DemoScreens.hpp
        include/Backends/OpenGLGLFW.hpp
        include/Utility/stb_image.h
        )

include_directories(${EXAMPLES_HEADERS})

#--------------------------------------------------------------------
# Create executable project
#--------------------------------------------------------------------
if (APPLE)
    #Get a list of all of the files in the data-folder
    file(GLOB_RECURSE RES_SOURCES "${CMAKE_SOURCE_DIR}/_Resources/Resources/*")
    add_executable(${PROJECT_NAME} MACOSX_BUNDLE ${EXAMPLES_SOURCES} ${RES_SOURCES})

    #individually set the file's path properties
    foreach(RES_FILE ${RES_SOURCES})

        #Get the relative path from the res folder to the particular file
        file(RELATIVE_PATH RES_PATH "${CMAKE_SOURCE_DIR}/_Resources" ${RES_FILE})

        get_filename_component(RES_PATH_BASEDIR ${RES_PATH} DIRECTORY)
        message("RES_PATH_BASEDIR: " ${RES_PATH_BASEDIR})
        #Set it's location inside the app package (under Resources)
        set_property(SOURCE ${RES_FILE} PROPERTY MACOSX_PACKAGE_LOCATION ${RES_PATH_BASEDIR})

    endforeach(RES_FILE)
else ()
    add_executable(${PROJECT_NAME} ${EXAMPLES_SOURCES} ${EXAMPLES_HEADERS})
endif ()

#--------------------------------------------------------------------
# Options & Definitions
#--------------------------------------------------------------------
target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_include_directories(${PROJECT_NAME} PRIVATE ${PROJECT_SOURCE_DIR}/src)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/LinaVG/include)

if (APPLE)
    include(FindPNG)
    include_directories(${PNG_INCLUDE_DIRS})
else ()
    # on windows MinGW64/Linux, use pkg-config to find libpng
    find_package(PkgConfig REQUIRED)
    message("build system is ${CMAKE_SYSTEM_NAME}")
    if (MSYS)
        set(ENV{PKG_CONFIG_PATH} "/mingw64/lib/pkgconfig")
    endif()
    pkg_check_modules(LIBPNG REQUIRED libpng)
    include_directories(${LIBPNG_INCLUDE_DIRS})
endif ()

include(FindBZip2)
if (BZIP2_FOUND)
    target_link_libraries(${PROJECT_NAME}
            PUBLIC BZip2::BZip2
            )
endif ()

if (APPLE)
    find_library(COCOA_LIBRARY Cocoa)
    find_library(OPENGL_LIBRARY OpenGL)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(COREVIDEO_LIBRARY CoreVideo)
    set(EXTRA_LIBS
            ${COCOA_LIBRARY}
            ${OPENGL_LIBRARY}
            ${IOKIT_LIBRARY}
            ${COREVIDEO_LIBRARY}
			glfw
            )	
endif ()

if (WIN32)
    include(${CMAKE_SOURCE_DIR}/_Dependencies/glfw/link.cmake)
    link_glfw()
	set(EXTRA_LIBS
		gdi32
	)
else ()
    # this imply add_library(glfw)
    find_package(glfw3 REQUIRED)
    if (NOT glfw3_FOUND)
        message("Error: glfw3 not found!")
    endif ()
endif ()
# print all cmake variables
#get_cmake_property(_variableNames VARIABLES)
#foreach (_variableName ${_variableNames})
#    message(STATUS "${_variableName}: ${${_variableName}}")
#endforeach()

#--------------------------------------------------------------------
# Links
#--------------------------------------------------------------------
target_link_libraries(${PROJECT_NAME}
        PUBLIC Lina::VG
        PUBLIC ${PNG_LIBRARIES}
        PUBLIC ${LIBPNG_LIBRARIES}
        PUBLIC ${EXTRA_LIBS}
        PUBLIC libglad_static
        PUBLIC ${FREETYPE2_LINK_LIBRARIES}
        )

# on Apple platform, use macos_bundle, otherwise copy resource
if (NOT APPLE)
    add_custom_command(
            TARGET ${PROJECT_NAME}
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/_Resources/Resources" "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/")
    add_custom_command(
            TARGET ${PROJECT_NAME}
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_SOURCE_DIR}/_Resources/Resources" "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIGURATION>/")
endif ()
#--------------------------------------------------------------------
# Folder structuring in visual studio
#--------------------------------------------------------------------
if (MSVC_IDE)
    foreach (source IN LISTS EXAMPLES_HEADERS EXAMPLES_SOURCES)
        get_filename_component(source_path "${source}" PATH)
        string(REPLACE "${EXAMPLES_SOURCE_DIR}" "" relative_source_path "${source_path}")
        string(REPLACE "/" "\\" source_path_msvc "${relative_source_path}")
        source_group("${source_path_msvc}" FILES "${source}")
    endforeach ()
endif ()
