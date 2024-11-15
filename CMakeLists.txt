cmake_minimum_required(VERSION 3.15)

# Project name and language settings
project(CircuitCalculator LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Add raylib
set(RAYLIB_VERSION 4.5.0)
find_package(raylib ${RAYLIB_VERSION} QUIET) 
if (NOT raylib_FOUND)
  include(FetchContent)
  FetchContent_Declare(
    raylib
    DOWNLOAD_EXTRACT_TIMESTAMP OFF
    URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
  )
  FetchContent_GetProperties(raylib)
  if (NOT raylib_POPULATED)
    FetchContent_Populate(raylib)
    set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    add_subdirectory(${raylib_SOURCE_DIR} ${raylib_BINARY_DIR})
  endif()
endif()

# Add libcurl (via vcpkg)
find_package(CURL REQUIRED)

# Add nlohmann-json (via vcpkg)
find_package(nlohmann_json CONFIG REQUIRED)

# Define the source directory
set(SRC_DIR ${CMAKE_SOURCE_DIR}/src)

# Automatically collect all source and header files
file(GLOB_RECURSE SOURCES "${SRC_DIR}/*.cpp" "${SRC_DIR}/*.hpp")

# Define the executable and include the collected files
add_executable(${PROJECT_NAME} ${SOURCES})

# Include the src directory for header file lookup
target_include_directories(${PROJECT_NAME} PRIVATE ${SRC_DIR})

# Link libraries
target_link_libraries(${PROJECT_NAME} PRIVATE 
    raylib 
    CURL::libcurl
    nlohmann_json::nlohmann_json
)

# Special settings for platforms
if (${PLATFORM} STREQUAL "Web")
    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".html")
endif()

if (APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE 
        "-framework IOKit"
        "-framework Cocoa"
        "-framework OpenGL"
    )
endif()

# Add definitions for Windows to avoid Raylib-specific warnings
if (WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()
