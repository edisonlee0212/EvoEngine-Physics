cmake_minimum_required(VERSION 3.17)

project(
	EvoEngine-Physics
	VERSION 0.1
	DESCRIPTION "An implementation of MLVQ library in CUDA with OptiX ray tracer."
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

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

# ------------------------------------------------------------------
# Physics - code
# ------------------------------------------------------------------

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
	)

if (WIN32)
	if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
		file(GLOB PHYSX_LIBS "3rdParty/physx/win.x86_64.vc143.mt/release/*.lib")
	else()
		file(GLOB PHYSX_LIBS "3rdParty/physx/win.x86_64.vc143.mt/debug/*.lib")
	endif ()
	target_link_libraries(evoengine-physics
			${PHYSX_LIBS}
			)
	# Compiler settings for Windows platform
	set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
elseif (UNIX AND NOT APPLE)
	if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
		file(GLOB PHYSX_LIBS "3rdParty/physx/linux.clang/release/*.a")
		set(CMAKE_CXX_FLAGS_RELEASE "-O3")
	elseif (CMAKE_BUILD_TYPE MATCHES "[Dd]ebug")
		file(GLOB PHYSX_LIBS "3rdParty/physx/linux.clang/debug/*.a")
		set(CMAKE_CXX_FLAGS_DEBUG "-g")
	endif ()
	target_link_libraries(evoengine
		${PHYSX_LIBS}
		)	

elseif (APPLE)
endif ()

# ------------------------------------------------------------------
# Copy Internal resources
# ------------------------------------------------------------------


if (NOT ${CMAKE_BINARY_DIR} STREQUAL ${PROJECT_BINARY_DIR})
	set(EVOENGINE_PHYSICS_INCLUDES ${LOCAL_EVOENGINE_PHYSICS_INCLUDES} PARENT_SCOPE)
	set(LOCAL_EVOENGINE_PHYSICS ${LOCAL_EVOENGINE_PHYSICS_PCH} PARENT_SCOPE)
endif ()

# ------------------------------------------------------------------
# Copy DLLs
# ------------------------------------------------------------------
if (WIN32)
    if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
        file(GLOB PHYSX_DLLS "3rdParty/physx/win.x86_64.vc143.mt/release/*.dll")
    elseif (CMAKE_BUILD_TYPE MATCHES "[Dd]ebug")
        file(GLOB PHYSX_DLLS "3rdParty/physx/win.x86_64.vc143.mt/debug/*.dll")
    endif ()
elseif (UNIX AND NOT APPLE)
    if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
        file(GLOB PHYSX_DLLS "3rdParty/physx/linux.clang/release/*.so")
    elseif (CMAKE_BUILD_TYPE MATCHES "[Dd]ebug")
        file(GLOB PHYSX_DLLS "3rdParty/physx/linux.clang/debug/*.so")
    endif ()
endif ()

foreach (PHYSX_DLL ${PHYSX_DLLS})
    add_custom_command(
            TARGET evoengine-physics
            POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy
            ${PHYSX_DLL}
            ${CMAKE_BINARY_DIR}
    )
endforeach ()