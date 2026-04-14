cmake_minimum_required(VERSION 3.20)

# ==============================
# Project definition
# ==============================
project(
    MyProject
    VERSION 1.0.0
    DESCRIPTION "General C++ project template"
    LANGUAGES CXX
)

# ==============================
# Global settings
# ==============================
# Pick your C++ standard here
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)   # OFF = use -std=c++17 instead of gnu++17

# Export compile_commands.json for IDEs, clangd, etc.
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Default build type if none is specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Build type" FORCE)
endif()

message(STATUS "Project name: ${PROJECT_NAME}")
message(STATUS "Project version: ${PROJECT_VERSION}")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")

# ==============================
# Options you may want to toggle
# ==============================
option(BUILD_SHARED_LIBS "Build shared libraries instead of static ones" OFF)
option(ENABLE_WARNINGS "Enable compiler warnings" ON)
option(ENABLE_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)
option(ENABLE_TESTS "Build tests" OFF)
option(ENABLE_SANITIZERS "Enable sanitizers for debug builds" OFF)
option(ENABLE_LTO "Enable link time optimization" OFF)

# ==============================
# Source files
# Replace these with your files
# ==============================
set(PROJECT_SOURCES
    src/main.cpp
    src/app.cpp
)

set(PROJECT_HEADERS
    include/app.hpp
)

# ==============================
# Main executable
# ==============================
add_executable(${PROJECT_NAME}
    ${PROJECT_SOURCES}
    ${PROJECT_HEADERS}
)

# Include directories
target_include_directories(${PROJECT_NAME}
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# ==============================
# Compile features
# ==============================
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)

# ==============================
# Warnings
# ==============================
if(ENABLE_WARNINGS)
    if(MSVC)
        target_compile_options(${PROJECT_NAME} PRIVATE /W4)
        if(ENABLE_WARNINGS_AS_ERRORS)
            target_compile_options(${PROJECT_NAME} PRIVATE /WX)
        endif()
    else()
        target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall
            -Wextra
            -Wpedantic
        )
        if(ENABLE_WARNINGS_AS_ERRORS)
            target_compile_options(${PROJECT_NAME} PRIVATE -Werror)
        endif()
    endif()
endif()

# ==============================
# Debug / Release-specific settings
# ==============================
if(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:DEBUG>
        $<$<CONFIG:Release>:NDEBUG>
    )
else()
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:-O0 -g>
        $<$<CONFIG:Release>:-O3>
    )
endif()

# ==============================
# Sanitizers (great for debugging)
# GCC/Clang only
# ==============================
if(ENABLE_SANITIZERS AND NOT MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:-fsanitize=address,undefined>
    )
    target_link_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:-fsanitize=address,undefined>
    )
endif()

# ==============================
# Link Time Optimization (LTO)
# ==============================
include(CheckIPOSupported)
check_ipo_supported(RESULT ipo_supported OUTPUT ipo_error)
if(ENABLE_LTO AND ipo_supported)
    set_property(TARGET ${PROJECT_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
elseif(ENABLE_LTO)
    message(WARNING "IPO/LTO not supported: ${ipo_error}")
endif()

# ==============================
# Platform-specific definitions
# ==============================
if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_WINDOWS)
elseif(APPLE)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_MACOS)
elseif(UNIX)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_LINUX)
endif()

# ==============================
# Example dependency
# Uncomment when needed
# ==============================
# find_package(OpenCV REQUIRED)
# target_link_libraries(${PROJECT_NAME} PRIVATE OpenCV::OpenCV)

# ==============================
# Example internal library
# Uncomment if you split code into a library
# ==============================
# add_library(my_core
#     src/core.cpp
#     include/core.hpp
# )
#
# target_include_directories(my_core PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
# target_compile_features(my_core PUBLIC cxx_std_17)
#
# target_link_libraries(${PROJECT_NAME} PRIVATE my_core)

# ==============================
# Install rules
# ==============================
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
)

# ==============================
# Tests
# ==============================
if(ENABLE_TESTS)
    enable_testing()

    # Example:
    # add_executable(test_app tests/test_app.cpp)
    # target_link_libraries(test_app PRIVATE my_core)
    # add_test(NAME test_app COMMAND test_app)
endif()
