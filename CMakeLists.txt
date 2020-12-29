include(CheckIncludeFile)
set(SOURCES src/nfd_common.c)

macro(REQUIRE_INCLUDE_FILE path name)
    CHECK_INCLUDE_FILE(${path} ${name})
    if (NOT ${name})
        message(FATAL_ERROR "${path} not found")
    endif ()
endmacro()

# add specific implementations
if (WIN32)
    REQUIRE_INCLUDE_FILE(windows.h HAS_WINDOWS)
    list(APPEND SOURCES src/nfd_win.cpp)
elseif (APPLE)
    REQUIRE_INCLUDE_FILE(AppKit/AppKit.h HAS_APPKIT)
    list(APPEND SOURCES src/nfd_cocoa.m)
elseif (UNIX)
    list(APPEND SOURCES src/nfd_gtk.c)
elseif (UNIX)
    message(FATAL_ERROR "Cannot detect your system, please report to https://github.com/aarcangeli/nativefiledialog-cmake/issues")
endif ()

add_library(nativefiledialog ${SOURCES})

target_include_directories(nativefiledialog PUBLIC src/include)

if (UNIX AND NOT APPLE)
    # GTK3 configuration adapted from https://gist.github.com/fracek/3323924

    # Use the package PkgConfig to detect GTK+ headers/library files
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

    # Setup CMake to use GTK+, tell the compiler where to look for headers
    # and to the linker where to look for libraries
    target_include_directories(nativefiledialog PRIVATE ${GTK3_INCLUDE_DIRS})
    target_link_directories(nativefiledialog PRIVATE ${GTK3_LIBRARY_DIRS})

    # Add other flags to the compiler
    target_compile_options(nativefiledialog PRIVATE ${GTK3_CFLAGS_OTHER})

    # Link the target to the GTK+ libraries
    target_link_libraries(nativefiledialog PRIVATE ${GTK3_LIBRARIES})
endif()
