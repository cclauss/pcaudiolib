cmake_minimum_required(VERSION 3.12)
project(pcaudiolib VERSION 1.3)

set(SRC_DIR "src")

# Add configuration file
configure_file(config.h.in config.h)

include_directories("${CMAKE_CURRENT_SOURCE_DIR}/src/include")