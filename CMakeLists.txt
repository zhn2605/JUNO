# Minimum CMake version
cmake_minimum_required(VERSION 3.15)

# Project name and language
project(JUNO VERSION 1.0
        DESCRIPTION "The rabbit"
        LANGUAGES C CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Options for build configuration
option(ENABLE_AUDIO "Enable audio support" ON)

# external libs
set(EXTERN_LIBS_DIR external)
# add glm as a subdir
add_subdirectory(${EXTERN_LIBS_DIR}/glm-master)
# manual glad stuff
set(GLAD_SOURCE_DIRS)

# WINDOWS BULLSHIT
if(WIN32)
    set(WIN32_LIBS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/win32_libs)
        
    # SDL2
    set(SDL2_DIR ${WIN32_LIBS_DIR}/SDL2-2.30.9)
    set(SDL2_INCLUDE_DIRS ${SDL2_DIR}/include)
    set(SDL2_LIBRARY_DIRS ${SDL2_DIR}/lib/x64)

    # Assimp
    set(ASSIMP_DIR ${WIN32_LIBS_DIR}/Assimp)
    set(ASSIMP_INCLUDE_DIRS ${ASSIMP_DIR}/include)
    set(ASSIMP_LIBRARY_DIRS ${ASSIMP_DIR}/lib/x64)

    set(ASSIMP_LINK_TARGET assimp-vc143-mt)
else()
    find_package(SDL2 REQUIRED)
    find_package(assimp REQUIRED)

    set(ASSIMP_LINK_TARGET assimp)
endif()

# find all source files
file(GLOB_RECURSE SOURCES 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.c
)

# manually add glad
list(APPEND SOURCES ${EXTERN_LIBS_DIR}/glad/src/glad.c)

add_executable(
    ${PROJECT_NAME} 
    ${SOURCES} 
)

# add include directories
target_include_directories(${PROJECT_NAME}
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${EXTERN_LIBS_DIR}/
    # manually add glad again
    ${EXTERN_LIBS_DIR}/glad/include
)

# other libraries
target_include_directories(
    ${PROJECT_NAME} PRIVATE
    ${SDL2_INCLUDE_DIRS}
    ${ASSIMP_INCLUDE_DIRS}
)

target_link_directories(
    ${PROJECT_NAME} PRIVATE
    ${SDL2_LIBRARY_DIRS}
    ${ASSIMP_LIBRARY_DIRS}
)

target_link_libraries(
    ${PROJECT_NAME} PRIVATE
    SDL2main SDL2
    ${ASSIMP_LINK_TARGET}
    glm::glm
)

# find and link common dependencies
find_package(OpenGL REQUIRED)
target_link_libraries(${PROJECT_NAME} PRIVATE OpenGL::GL)

# install rules
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# set output directories
set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
)

# debug configurations
if(CMAKE_BUILD_TYPE STREQUAL Debug)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _DEBUG)
endif()

# copy DLLs (windows)
if(WIN32)
    # SDL2
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${SDL2_DIR}/lib/x64/SDL2.dll"
        $<TARGET_FILE_DIR:${PROJECT_NAME}>
    )
    
    # ASSIMP
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "${ASSIMP_DIR}/bin/x64/assimp-vc143-mt.dll"
        $<TARGET_FILE_DIR:${PROJECT_NAME}>
    )
endif()

# copy assets folder over
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    "${CMAKE_CURRENT_SOURCE_DIR}/assets"
    "$<TARGET_FILE_DIR:${PROJECT_NAME}>/assets"
)