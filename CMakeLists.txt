# Minimum CMake version
cmake_minimum_required(VERSION 3.15)

# Project name and language
project(JUNO VERSION 1.0
        DESCRIPTION "The rabbit"
        LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Options for build configuration
option(ENABLE_AUDIO "Enable audio support" ON)

# Find all source files
file(GLOB_RECURSE SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c"
)

# Find all header files
file(GLOB_RECURSE HEADERS
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
)

# Create the executable
add_executable(${PROJECT_NAME} ${SOURCES} &{HEADERS})

# Add include directories
target_include_directories(${PROJECT_NAME}
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# Platform-specific configurations
if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE WINDOWS_BUILD)
    # Windows-specific libraries
    target_link_libraries(${PROJECT_NAME} PRIVATE winmm)
elseif(APPLE)
    target_compile_definitions(${PROJECT_NAME} PRIVATE MACOS_BUILD)
    # MacOS-specific frameworks
    find_library(COCOA_LIBRARY Cocoa)
    target_link_libraries(${PROJECT_NAME} PRIVATE ${COCOA_LIBRARY})
elseif(UNIX)
    target_compile_definitions(${PROJECT_NAME} PRIVATE LINUX_BUILD)
    # Linux-specific libraries
    find_package(X11 REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE X11)
endif()

# Find and link common dependencies
find_package(OpenGL REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL)

# Audio support
# if(ENABLE_AUDIO)
#     find_package(OpenAL REQUIRED)
#     target_link_libraries(${PROJECT_NAME} PRIVATE OpenAL::OpenAL)
#     target_compile_definitions(${PROJECT_NAME} PRIVATE ENABLE_AUDIO_SUPPORT)
# endif()
#

# Install rules
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# Set output directories
set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
)

# Debug configurations
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(${PROJECT_NAME} PRIVATE _DEBUG)
endif()