cmake_minimum_required(VERSION 4.4.2)

project(vimpc VERSION 0.09.3 LANGUAGES CXX)

# clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Options
option(ENABLE_LYRICS "Enable lyric fetching" ON)
option(ENABLE_TAGLIB "Enable TagLib support" ON)
option(ENABLE_TEST "Build tests [NOT IMPLEMENTED]" OFF)
option(ENABLE_PROFILER "Enable profiler support [NOT IMPLEMENTED]" OFF)

set(PACKAGE_URL "https://github.com/sdharme/vimpc")

# Sources
add_executable(${PROJECT_NAME}
    src/algorithm.cpp
    src/buffer/browse.cpp
    src/buffer/directory.cpp
    src/buffer/library.cpp
    src/buffers.cpp
    src/clientstate.cpp
    src/errorcodes.cpp
    src/events.cpp
    src/main.cpp
    src/mode/inputmode.cpp
    src/mode/search.cpp
    src/mode/normal.cpp
    src/mode/command.cpp
    src/output.cpp
    src/player.cpp
    src/regex.cpp
    src/screen.cpp
    src/song.cpp
    src/vimpc.cpp
    src/window/browsewindow.cpp
    src/window/console.cpp
    src/window/debug.cpp
    src/window/error.cpp
    src/window/infowindow.cpp
    src/window/librarywindow.cpp
    src/window/listwindow.cpp
    src/window/modewindow.cpp
    src/window/outputwindow.cpp
    src/window/pagerwindow.cpp
    src/window/playlistwindow.cpp
    src/window/songwindow.cpp
    src/window/window.cpp
    src/window/windowselector.cpp
    src/window/help.cpp
    src/window/selectwindow.cpp
    src/window/directorywindow.cpp
    src/window/scrollwindow.cpp
    src/mpdclient.cpp
    src/colours.cpp
    src/settings.cpp
)

# Generate config.h
include(CheckIncludeFileCXX)

check_include_file_cxx("execinfo.h" HAVE_EXECINFO_H)
check_include_file_cxx("sys/ioctl.h" HAVE_SYS_IOCTL_H)

include(GNUInstallDirs)

set(HELP_DIRECTORY
    "${CMAKE_INSTALL_FULL_DATAROOTDIR}/doc/${PROJECT_NAME}"
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/src/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/src/config.h"
)

# Copy help.txt
add_custom_target(copy_help ALL
    COMMAND ${CMAKE_COMMAND} -E make_directory
            "${CMAKE_CURRENT_BINARY_DIR}/doc"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_SOURCE_DIR}/doc/help.txt"
            "${CMAKE_CURRENT_BINARY_DIR}/doc/help.txt"
)
add_dependencies(${PROJECT_NAME} copy_help)

# Required dependencies
cmake_pkg_config(IMPORT ncursesw REQUIRED)
cmake_pkg_config(IMPORT libpcre REQUIRED)
cmake_pkg_config(IMPORT libmpdclient REQUIRED)

# Lyrics fetching support
if(ENABLE_LYRICS)
    target_sources(${PROJECT_NAME} PRIVATE
        src/lyricsfetcher.cpp
        src/lyricsloader.cpp
        src/window/lyricswindow.cpp
    )

    cmake_pkg_config(IMPORT libcurl REQUIRED)

    target_compile_definitions(${PROJECT_NAME} PRIVATE
        LYRICS_SUPPORT=1
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        PkgConfig::libcurl
    )
endif()

# TagLib support
if(ENABLE_TAGLIB)
    cmake_pkg_config(IMPORT taglib REQUIRED)

    target_compile_definitions(${PROJECT_NAME} PRIVATE
        TAG_SUPPORT=1
        HAVE_TAGLIB_H=1
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE
        PkgConfig::taglib
    )
endif()

# Target configuration
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_BINARY_DIR}/src
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    PkgConfig::ncursesw
    PkgConfig::libpcre
    PkgConfig::libmpdclient
)

# Installation
install(TARGETS ${PROJECT_NAME} RUNTIME
    DESTINATION "${CMAKE_INSTALL_BINDIR}"
)

install(FILES LICENSE doc/help.txt doc/vimpcrc.example
    DESTINATION "${CMAKE_INSTALL_DATADIR}/doc/${PROJECT_NAME}"
)

install(FILES doc/vimpc.1
    DESTINATION "${CMAKE_INSTALL_MANDIR}/man1"
)
