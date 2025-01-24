cmake_minimum_required(VERSION 3.12)
project(pcaudiolib VERSION 1.3)

# Set the source directory
set(SRC_DIR "src")

# Add configuration file
configure_file(config.h.in config.h)

# Include the source directory
include_directories("${CMAKE_CURRENT_SOURCE_DIR}/src/include")

# Define options for audio backends
option(WITH_ALSA "Enable ALSA audio output support" OFF)
option(WITH_PULSEAUDIO "Enable PulseAudio audio output support" OFF)
option(WITH_COREAUDIO "Enable CoreAudio audio output support (macOS)" ON)
option(WITH_OSS "Enable OSS audio output support" OFF)

# Collect all source files in the src directory
file(GLOB_RECURSE SOURCES "${SRC_DIR}/*.c")


# Exclude platform-specific files based on the OS

if(NOT WIN32)
    list(FILTER SOURCES EXCLUDE REGEX "windows.c")
endif()

if(NOT WITH_ALSA OR NOT UNIX)
    list(FILTER SOURCES EXCLUDE REGEX "alsa.c")
endif()

if(NOT WITH_COREAUDIO OR NOT APPLE)
    list(FILTER SOURCES EXCLUDE REGEX "coreaudio.c")
    list(FILTER SOURCES EXCLUDE REGEX "TPCircularBuffer")
endif()

if(NOT WITH_OSS OR NOT UNIX)
    list(FILTER SOURCES EXCLUDE REGEX "oss.c")
endif()

if(NOT WITH_PULSEAUDIO OR NOT UNIX)
    list(FILTER SOURCES EXCLUDE REGEX "pulseaudio.c")
endif()

# ALSA Integration
if(WITH_ALSA)
    find_package(ALSA REQUIRED)
    if(ALSA_FOUND)
        target_include_directories(${PROJECT_NAME} PRIVATE ${ALSA_INCLUDE_DIRS})
        target_link_libraries(${PROJECT_NAME} PRIVATE ${ALSA_LIBRARIES})
        add_definitions(-DHAVE_ALSA_ASOUNDLIB_H)
        message(STATUS "ALSA support enabled and libraries linked.")
    else()
        message(WARNING "ALSA not found. Disabling ALSA support.")
        set(WITH_ALSA OFF)
    endif()
endif()

# Placeholder for other backends (future work)
if(WITH_PULSEAUDIO)
    message(STATUS "PulseAudio support is not yet implemented.")
endif()

if(WITH_COREAUDIO)
    message(STATUS "CoreAudio support is not yet implemented.")
endif()

if(WITH_OSS)
    message(STATUS "OSS support is not yet implemented.")
endif()

# Add sources and create a static library
add_library(${PROJECT_NAME} SHARED ${SOURCES})

# Add the configuration header to the target
target_include_directories(${PROJECT_NAME} PRIVATE "${CMAKE_CURRENT_BINARY_DIR}")

# Additional settings
set_target_properties(${PROJECT_NAME} PROPERTIES C_STANDARD 99)

# Output messages for clarity
message(STATUS "Project: ${PROJECT_NAME}")
message(STATUS "Version: ${PROJECT_VERSION}")
message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "CMake Generator: ${CMAKE_GENERATOR}")
message(STATUS "Options:")
message(STATUS "  WITH_ALSA: ${WITH_ALSA}")
message(STATUS "  WITH_PULSEAUDIO: ${WITH_PULSEAUDIO}")
message(STATUS "  WITH_COREAUDIO: ${WITH_COREAUDIO}")
message(STATUS "  WITH_OSS: ${WITH_OSS}")


install(TARGETS ${PROJECT_NAME} DESTINATION lib)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/include/ DESTINATION include)

# cmake -B build . -DCMAKE_INSTALL_PREFIX=_install 
# cmake --build build