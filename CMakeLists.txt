cmake_minimum_required(VERSION 3.7)

project(PCem)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/output/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/output/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/output/bin)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/includes/private)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/includes/public)

# TODO: This whole thing is jank, replace with a real way to determine cpu type
if(WIN32)
    if(${CMAKE_C_COMPILER} MATCHES ".*mingw32.*")
        set(CMAKE_SYSTEM_PROCESSOR i386)
    elseif(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "AMD64")
        set(CMAKE_SYSTEM_PROCESSOR x86_64)
    endif()
endif()

set(PCEM_CPU_TYPE ${CMAKE_SYSTEM_PROCESSOR})
# End of Jankness

if(${PCEM_CPU_TYPE} STREQUAL "i386")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -msse2")
	set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -msse2")
endif()

set(PCEM_VERSION_STRING "vNext" CACHE STRING "PCem Version String")
set(PCEM_DEFINES PCEM_VERSION_STRING="${PCEM_VERSION_STRING}")

message("PCem ${PCEM_VERSION_STRING}")
message("***************************************************************")
message("OS Building For: ${CMAKE_SYSTEM_NAME}")
message("System Processor: ${PCEM_CPU_TYPE}")
message("***************************************************************")

option(USE_NETWORKING "Build PCem with networking support" OFF)
message("Networking Support: ${USE_NETWORKING}")

option(USE_ALSA "Build PCem with MIDI output using ALSA" OFF)
message("ALSA MIDI Support: ${USE_ALSA}")

message("***************************************************************")

set(OpenGL_GL_PREFERENCE GLVND)
find_package(SDL2 REQUIRED)
include_directories(${SDL2_INCLUDE_DIRS})

find_package(OpenAL REQUIRED)
include_directories(${OPENAL_INCLUDE_DIR})

if(WIN32)
    set(wxWidgets_CONFIGURATION mswu)
endif()
find_package(wxWidgets REQUIRED COMPONENTS core base xrc)
include(${wxWidgets_USE_FILE})

if(USE_ALSA)
    find_package(ALSA REQUIRED)
    include_directories(${ALSA_INCLUDE_DIRS})
endif()

find_package(OpenGL REQUIRED)

add_subdirectory(src)
