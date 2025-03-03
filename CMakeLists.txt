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
cmake_minimum_required (VERSION 3.6)
project(LinaVG)
set(CMAKE_CXX_STANDARD 17)

#--------------------------------------------------------------------
# Download dependencies
#--------------------------------------------------------------------

if(NOT EXISTS ${CMAKE_SOURCE_DIR}/_Dependencies)
	include(CMake/GitUtils.cmake)
	git_clone(
        	PROJECT_NAME    _Dependencies
        	GIT_URL         https://github.com/inanevin/LinaDependencies.git
        	GIT_BRANCH      dev_linavg
        	DIRECTORY 	${CMAKE_SOURCE_DIR})
endif()

#--------------------------------------------------------------------
# Options & Definitions
#--------------------------------------------------------------------

option(LINAVG_BUILD_EXAMPLES "Builds example backend projects." ON)
option(LINAVG_BACKEND_GL "Enables OpenGL Backend." ON)
option(LINAVG_TEXT_SUPPORT "Enables FreeType text support. Requires you to link to FreeType library." ON)

if(LINAVG_BACKEND_GL)
	add_compile_definitions(LINAVG_BACKEND_GL)
endif()

if(LINAVG_BUILD_EXAMPLES)
	add_compile_definitions(LINAVG_TEXT_SUPPORT)
else()
	if(LINAVG_TEXT_SUPPORT)
		add_compile_definitions(LINAVG_TEXT_SUPPORT)
	endif()
endif()


set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_SUPPRESS_REGENERATION true)

if (MSVC)
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
endif (MSVC)

#--------------------------------------------------------------------
# Set sources
#--------------------------------------------------------------------

set(LinaVG_SOURCES 

    # Core
    include/LinaVG.hpp
    include/Utility/Utility.hpp
    include/Core/Renderer.hpp
    include/Core/Text.hpp
    include/Core/Drawer.hpp
    include/Core/Backend.hpp
    include/Core/Common.hpp
    include/Core/Math.hpp
    include/Backends/BaseBackend.hpp
)

if(LINAVG_BACKEND_GL)
    set(LinaVG_BackendHPP include/Backends/GL/GLBackend.hpp)
    set(LinaVG_BackendCPP src/Backends/GL/GLBackend.cpp)
else()
    set(LinaVG_BackendHPP)
    set(LinaVG_BackendCPP)
endif()

set(LinaVG_HEADERS

    # Core
    src/Utility/Utility.cpp
    src/Core/Renderer.cpp
    src/Core/Text.cpp
    src/Core/Backend.cpp
    src/Core/Drawer.cpp
    src/Core/Common.cpp
    src/Core/Math.cpp
)

#--------------------------------------------------------------------
# Create executable project
#--------------------------------------------------------------------
add_library(${PROJECT_NAME} ${LinaVG_SOURCES} ${LinaVG_HEADERS} ${LinaVG_BackendHPP} ${LinaVG_BackendCPP})
add_library(Lina::VG ALIAS ${PROJECT_NAME}) 

#--------------------------------------------------------------------
# Set include directories
#--------------------------------------------------------------------

target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_include_directories(${PROJECT_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/_Dependencies/glad/include)

target_compile_definitions(${PROJECT_NAME} PUBLIC LINAVG_VERSION_MAJOR=1)
target_compile_definitions(${PROJECT_NAME} PUBLIC LINAVG_VERSION_MINOR=3)
target_compile_definitions(${PROJECT_NAME} PUBLIC LINAVG_VERSION_PATCH=5)

#--------------------------------------------------------------------
# Subdirectories & linking
#--------------------------------------------------------------------

if(LINAVG_BACKEND_GL)
	add_subdirectory(_Dependencies/glad)
endif()


if(LINAVG_BUILD_EXAMPLES OR LINAVG_TEXT_SUPPORT)
	if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
		find_package(PkgConfig REQUIRED)
		pkg_check_modules(FREETYPE2 REQUIRED freetype2)
		if (NOT FREETYPE2_FOUND)
			message("FREETYPE2 not found! ")
		endif()
		include_directories(${FREETYPE2_INCLUDE_DIRS})
	else()
		include(_Dependencies/freetype/link.cmake)
		link_freetype()
	endif()
endif()

if(LINAVG_BUILD_EXAMPLES)
	#--------------------------------------------------------------------
	# Folder structuring in visual studio
	#--------------------------------------------------------------------
	if(MSVC_IDE)
		foreach(source IN LISTS LinaVG_HEADERS LinaVG_SOURCES LinaVG_BackendHPP LinaVG_BackendCPP)
			get_filename_component(source_path "${source}" PATH)
			string(REPLACE "${LinaVG_SOURCE_DIR}" "" relative_source_path "${source_path}")
			string(REPLACE "/" "\\" source_path_msvc "${relative_source_path}")
			source_group("${source_path_msvc}" FILES "${source}")
		endforeach()
	endif()
	add_subdirectory(Examples)
	if(MSVC_IDE)
		set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT Examples)
	endif()
endif()
