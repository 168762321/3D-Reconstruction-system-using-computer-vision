# Copyright (c) 2021-2022, HUALU MEIDA CORPORATION.  All rights reserved.
#
# HUALU MEIDA CORPORATION and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from HUALU MEIDA CORPORATION is strictly prohibited.

cmake_minimum_required(VERSION 3.10)

PROJECT(holemap
	VERSION 1.0
	DESCRIPTION "3d reconstruction using computer vision"
	LANGUAGES C CXX CUDA
)
set(HOLEMAP_VERSION "${CMAKE_PROJECT_VERSION}")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

###############################################################################
# Build type and C++ compiler setup
###############################################################################

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
set(CMAKE_CXX_STANDARD 17)  #修改c++版本
set(CMAKE_CXX_EXTENSIONS OFF)

###############################################################################
# CUDA compiler setup
###############################################################################

set(CMAKE_CUDA_STANDARD 14)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_EXTENSIONS OFF)
set(CUDA_LINK_LIBRARIES_KEYWORD PUBLIC)

list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-mf16c")
list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-Wno-float-conversion")
list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-fno-strict-aliasing")
list(APPEND CUDA_NVCC_FLAGS "-fPIC")
set(CUDA_TOOLKIT_ROOT_DIR /opt/cuda/targets/x86_64-linux)

list(APPEND CUDA_NVCC_FLAGS "--extended-lambda")
list(APPEND CUDA_NVCC_FLAGS "--expt-relaxed-constexpr")

###############################################################################
# Dependencies (ordered alphabetically)
###############################################################################

include_directories("dependencies")

########
# GLFW #
########
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL " " FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL " " FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL " " FORCE)
set(GLFW_BUILD_INSTALL OFF CACHE BOOL " " FORCE)
set(GLFW_INSTALL OFF CACHE BOOL " " FORCE)
set(GLFW_USE_CHDIR OFF CACHE BOOL " " FORCE)
set(BUILD_SHARED_LIBS ON CACHE BOOL " " FORCE)

add_subdirectory(dependencies/glfw)

set_target_properties(glfw PROPERTIES EXCLUDE_FROM_ALL 1 EXCLUDE_FROM_DEFAULT_BUILD 1)

mark_as_advanced(
	GLFW_BUILD_DOCS GLFW_BUILD_EXAMPLES GLFW_BUILD_INSTALL GLFW_BUILD_TESTS
	GLFW_DOCUMENT_INTERNALS GLFW_INSTALL GLFW_USE_CHDIR GLFW_USE_MENUBAR
	GLFW_USE_OSMESA GLFW_VULKAN_STATIC GLFW_USE_RETINA GLFW_USE_MIR
	BUILD_SHARED_LIBS USE_MSVC_RUNTIME_LIBRARY_DLL
)

include_directories("dependencies/glfw/include")

#############
# GL3W/GLEW #
#############

find_package(GLEW REQUIRED)
include_directories(${GLEW_INCLUDE_DIRS})
set(GL_LIBRARIES GL ${GLEW_LIBRARIES} $<TARGET_OBJECTS:glfw_objects>)

set(GL_SOURCES
	${GL_SOURCES}
	dependencies/imgui/ImGuizmo.cpp
	dependencies/imgui/imgui.cpp
	dependencies/imgui/imgui_impl_glfw.cpp
	dependencies/imgui/imgui_impl_opengl3.cpp
	dependencies/imgui/imgui_draw.cpp
	dependencies/imgui/imgui_tables.cpp
	dependencies/imgui/imgui_widgets.cpp
	dependencies/imgui/imgui_demo.cpp
	
)

##########
# OpenMP #
##########
find_package(OpenMP)
if (OPENMP_FOUND)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

###############################################################################
# Program
###############################################################################

include_directories("include")

set(SOURCES
	${GL_SOURCES}
	# src/main.cpp
	# src/demo.cpp
)

add_library(hole STATIC ${SOURCES})
target_link_libraries(hole PUBLIC ${GL_LIBRARIES} -ldl)

add_executable(${CMAKE_PROJECT_NAME} src/main.cpp)
# add_executable(${CMAKE_PROJECT_NAME} src/demo.cpp)
target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC hole -ldl)
