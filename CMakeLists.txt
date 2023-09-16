cmake_minimum_required(VERSION 3.17)

project(
        EvoEngine-Physics
        VERSION 0.1
        DESCRIPTION "An implementation of MLVQ library in CUDA with OptiX ray tracer."
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# ------------------------------------------------------------------
# Options
# ------------------------------------------------------------------
option(BUILD_PHYSICS "Build Physics" ON)

# ------------------------------------------------------------------
# Build types
# ------------------------------------------------------------------
# Set a default build type if none was specified
set(default_build_type "Release")

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
            STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            "Debug" "Release")
endif ()

if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
    set(DEBUG_STATUS NDEBUG)
elseif (CMAKE_BUILD_TYPE MATCHES "[Dd]ebug")
    set(DEBUG_STATUS _DEBUG)
endif ()

# ------------------------------------------------------------------
# Dependencies
# ------------------------------------------------------------------
# We need a CMAKE_DIR with some code to find external dependencies
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
set(EVOENGINE_PHYSICS_3RD_PARTY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty)
add_subdirectory(${EVOENGINE_PHYSICS_3RD_PARTY_DIR})

set(EVOENGINE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../EvoEngine)
if (NOT TARGET evoengine)
    add_subdirectory(${EVOENGINE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/EvoEngine)
endif ()
include_directories(${EVOENGINE_INCLUDES})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# ------------------------------------------------------------------
# Physics - code
# ------------------------------------------------------------------

if (BUILD_PHYSICS)
    file(GLOB LOCAL_EVOENGINE_PHYSICS_SOURCES "src/lib/*.cpp")
    add_library(evoengine-physics
        STATIC
        ${LOCAL_EVOENGINE_PHYSICS_SOURCES}
        )

    set(LOCAL_EVOENGINE_PHYSICS_PCH 
        ${EVOENGINE_PCH}
        ${CMAKE_CURRENT_SOURCE_DIR}/include/lib/evoengine-physics-pch.hpp
        )

    target_precompile_headers(evoengine-physics
        PRIVATE
        ${LOCAL_EVOENGINE_PHYSICS_PCH}
        )

    set(LOCAL_EVOENGINE_PHYSICS_INCLUDES
        ${EVOENGINE_PHYSICS_3RD_PARTY_INCLUDES}
        ${EVOENGINE_INCLUDES}
        ${CMAKE_CURRENT_SOURCE_DIR}/include/lib
        ${CMAKE_CURRENT_SOURCE_DIR}/include/lib/Physics
        ${CMAKE_CURRENT_BINARY_DIR}
        )

    target_include_directories(evoengine-physics
        PRIVATE
        ${LOCAL_EVOENGINE_PHYSICS_INCLUDES}
        )

    target_link_libraries(evoengine-physics
        evoengine
        )
    target_compile_definitions(evoengine-physics
            PRIVATE
            NOMINMAX
            HAS_PHYSICS
            )
endif ()

if (WIN32)
    # Compiler settings for Windows platform
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
elseif (UNIX AND NOT APPLE)
    if (${CMAKE_BUILD_TYPE} STREQUAL Release)
        set(CMAKE_CXX_FLAGS_RELEASE "-O3")
    elseif (${CMAKE_BUILD_TYPE} STREQUAL Debug)
        set(CMAKE_CXX_FLAGS_DEBUG "-g")
    endif ()
elseif (APPLE)
endif ()

# ------------------------------------------------------------------
# Copy Internal resources
# ------------------------------------------------------------------

if (BUILD_PHYSICS)
    if (NOT ${CMAKE_BINARY_DIR} STREQUAL ${PROJECT_BINARY_DIR})
        set(EVOENGINE_PHYSICS_INCLUDES ${LOCAL_EVOENGINE_PHYSICS_INCLUDES} PARENT_SCOPE)
    endif ()
endif ()