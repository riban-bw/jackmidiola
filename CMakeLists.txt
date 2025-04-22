cmake_minimum_required(VERSION 3.0)
# set(CMAKE_VERBOSE_MAKEFILE ON)

project(jackmidiola)

include(CheckIncludeFiles)
include(CheckLibraryExists)

link_directories(/usr/local/lib)

add_executable(jackmidiola midiola.cpp)
add_definitions(-Werror)
target_link_libraries(jackmidiola jack ola olacommon protobuf)

install(TARGETS jackmidiola
    DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
)

