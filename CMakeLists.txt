cmake_minimum_required (VERSION 3.1)   # First version with CMAKE_CXX_STANDARD.
project(proto3d)

set (CMAKE_CXX_STANDARD 11)

# Maximum warnings level minus some
set(CMAKE_CXX_FLAGS
  "${CMAKE_CXX_FLAGS} -g -Wall -pedantic -Wextra -Wno-unused-parameter -Wno-writable-strings")

# Extra flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CXX_EXTRA_FLAGS}")

# Passing -fno-exceptions to the compiler is not possible yet due to some
# reliance on STL.
OPTION(PROTO3D_EXCEPTIONS
  "Enable code that throws exceptions from proto3d" ON)
OPTION(PROTO3D_GLM
  "Include GLM headers and enable code using it" OFF)
OPTION(PROTO3D_STB_IMAGE
  "Compile stb_image.c and allow code using it" ON)
OPTION(PROTO3D_DEMOS
  "Also build the demos in demos/" ON)

# Add / to the include path
include_directories(${PROJECT_SOURCE_DIR})

# Add .cpp files to the static library
if(PROTO3D_STB_IMAGE)
  add_definitions(-DPROTO3D_USE_STB)
endif(PROTO3D_STB_IMAGE)

if(PROTO3D_EXCEPTIONS)
  add_definitions(-DPROTO3D_USE_EXCEPTIONS)
endif()

# Use pkg-config to find some libraries
include(FindPkgConfig)

# Find OpenGL
include(FindOpenGL)
if(OPENGL_FOUND)
  message("Great. OpenGL found.")
else()
  message(FATAL_ERROR "OpenGL not found.")
endif()

# Include glm
if(PROTO3D_GLM)
  # GLM is header-only
  include_directories(glm)
  add_definitions(-DPROTO3D_USE_GLM)
endif(PROTO3D_GLM)

# Set up library and include paths
#
# We should end up with these in compiler invocations:
# "-framework Cocoa -framework IOKit -framework CoreFoundation -framework CoreVideo -framework Carbon"
find_library(COCOA_FRAMEWORK Cocoa)
find_library(IOKIT_FRAMEWORK IOKit)
find_library(CORE_FOUNDATION_FRAMEWORK CoreFoundation)
find_library(CORE_VIDEO_FRAMEWORK CoreVideo)
find_library(CARBON_FRAMEWORK Carbon)
mark_as_advanced(COCOA_FRAMEWORK
                 IOKIT_FRAMEWORK
                 CORE_FOUNDATION_FRAMEWORK
                 CORE_VIDEO_FRAMEWORK
                 CARBON_FRAMEWORK)
list(APPEND APPLE_LIBRARIES "${COCOA_FRAMEWORK}"
                          "${IOKIT_FRAMEWORK}"
                          "${CORE_FOUNDATION_FRAMEWORK}"
                          "${CORE_VIDEO_FRAMEWORK}"
                          "${CARBON_FRAMEWORK}")

# Include demos/CMakeLists.txt after setting everything we need
if(PROTO3D_DEMOS)
  add_subdirectory(demos)
endif(PROTO3D_DEMOS)
