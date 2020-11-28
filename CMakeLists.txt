##
# This file is part of Menderer.
#
# Copyright 2019 Robert Maier, Technical University of Munich.
# For more information see <https://github.com/robmaier/menderer>.
# If you use this code, please cite the respective publications as
# listed on the above website.
#
# Menderer is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# Menderer is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with Menderer. If not, see <http://www.gnu.org/licenses/>.
##

#==============================================================================
# CMake file for Menderer project.
# Original Author: Robert Maier (robert.maier@tum.de)
# Contributer : Gyu Jin Choi (paganinist@gmail.com)
#==============================================================================

#==============================================================================
CMAKE_MINIMUM_REQUIRED(VERSION 3.2 FATAL_ERROR)
PROJECT(Menderer C CXX)

#==============================================================================
# CMake paths
# output path for generated executables
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
#==============================================================================

#==============================================================================
# Dependencies
#==============================================================================

# include header-only third-party libraries (Eigen, CLI11)
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/thirdparty/headeronly)

# OpenCV
FIND_PACKAGE(OpenCV 4.3.0 COMPONENTS core highgui imgproc REQUIRED)
IF(OPENCV_CORE_FOUND)
    INCLUDE_DIRECTORIES(${OpenCV_INCLUDE_DIRS})
    MESSAGE(STATUS "${OpenCV_LIBS}")
ENDIF()

# GLAD (build directly from third-party subdirectory)
ADD_SUBDIRECTORY("${PROJECT_SOURCE_DIR}/thirdparty/glad-0.1.34/")
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/thirdparty/glad-0.1.34/include)

# GLFW3 (build directly from third-party subdirectory)
SET(GLFW_BUILD_DOCS OFF CACHE BOOL "Build the GLFW docs")
SET(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "Build the GLFW example programs")
SET(GLFW_BUILD_TESTS OFF CACHE BOOL "Build the GLFW tests")
SET(GLFW_INSTALL OFF CHACHE BOOL "Generate installation target")
ADD_SUBDIRECTORY("${PROJECT_SOURCE_DIR}/thirdparty/glfw-3.3.2/")
INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/thirdparty/glfw-3.3.2/include)

#==============================================================================
# Build Menderer application
#==============================================================================

# ------------------------------------------------------------------------
# C++ flags
IF(CMAKE_COMPILER_IS_GNUCC)
    # Add -O0 to remove optimizations when using gcc
    SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -O0")
    SET(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -O0")
ENDIF()

# enforce C++11 standard
if(CMAKE_VERSION VERSION_LESS 3.1)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
else()
    IF(NOT DEFINED CMAKE_CXX_STANDARD OR CMAKE_CXX_STANDARD STREQUAL "98")
        SET(CMAKE_CXX_STANDARD 11)
    ENDIF()
    SET(CMAKE_CXX_STANDARD_REQUIRED ON)
ENDIF()

# ------------------------------------------------------------------------
# Application

# preprocessor define: source directory (e.g. for finding OpenGL shaders)
ADD_DEFINITIONS(-DAPP_SOURCE_DIR=${PROJECT_SOURCE_DIR})

# glob headers and source files
FILE(GLOB_RECURSE incs "${CMAKE_CURRENT_SOURCE_DIR}/" "include/*.h")
FILE(GLOB_RECURSE srcs "${CMAKE_CURRENT_SOURCE_DIR}/" "src/*.cpp")
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/include)
# group headers and source files
SOURCE_GROUP("Header Files" FILES ${incs})
SOURCE_GROUP("Source Files" FILES ${srcs})

# add executable
ADD_EXECUTABLE(${PROJECT_NAME} ${incs} ${srcs} )
TARGET_LINK_LIBRARIES(${PROJECT_NAME}
    ${OpenCV_LIBS}
    glfw
    glad
)
TARGET_COMPILE_OPTIONS(${PROJECT_NAME} PRIVATE -std=c++11)
