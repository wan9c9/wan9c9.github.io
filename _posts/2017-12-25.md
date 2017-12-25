---
layout: post
title: How to write Cmake files
---

Always specifiy minimum required version of cmake

cmake_minimum_required(VERSION 3.9)
You should declare a project. cmake says it is mandatory and it will define convenient variables PROJECT_NAME, PROJECT_VERSION and PROJECT_DESCRIPTION (this latter variable necessitate cmake 3.9):

project(mylib VERSION 1.0.1 DESCRIPTION "mylib description")
Declare a new library target. Please avoid use of file(GLOB ...). This feature does not provide attended mastery of compilation process. If you are lazy, copy-paste output of ls -1 sources/*.cpp :

add_library(mylib SHARED
    sources/animation.cpp
    sources/buffers.cpp
    [...]
)
Set VERSION property (optional but it is a good practice):

set_target_properties(mylib PROPERTIES VERSION ${PROJECT_VERSION})
You can also set SOVERSION to major number of VERSION. So libmylib.so.1 will be a symlink to libmylib.so.1.0.0.

set_target_properties(mylib PROPERTIES SOVERSION 1)
Declare public API of your library. This API will be installed for third-party application. It is a good practice to isolate it in your project tree (like placing it include/ directory). Notice that, private headers should not been installed and I strongly suggest to place them with sources files.

set_target_properties(mylib PROPERTIES PUBLIC_HEADER include/mylib.h)
If you work with subdirectories, it is not very convenient to include relative path like "../include/mylib.h". So, pass top directory in included directories:

target_include_directories(mylib PRIVATE .)
or

target_include_directories(mylib PRIVATE include)
target_include_directories(mylib PRIVATE src)
Create install rule for your library. I suggest to use variables CMAKE_INSTALL_*DIR defined in GNUInstallDirs:

include(GNUInstallDirs)
And declare files to install:

install(TARGETS mylib
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
You may also export a pkg-config file. This files allows third-party application to easily import your library:

with Makefile, see pkg-config
with Autotools, see PKG_CHECK_MODULES
with cmake, see pkg_check_modules
Create a template file named mylib.pc.in:

prefix=@CMAKE_INSTALL_PREFIX@
exec_prefix=@CMAKE_INSTALL_PREFIX@
libdir=${exec_prefix}/@CMAKE_INSTALL_LIBDIR@
includedir=${prefix}/@CMAKE_INSTALL_INCLUDEDIR@

Name: @PROJECT_NAME@
Description: @PROJECT_DESCRIPTION@
Version: @PROJECT_VERSION@

Requires:
Libs: -L${libdir} -lmylib
Cflags: -I${includedir}
In your CMakeLists.txt, add a rule to expand @ macros (@ONLY ask to cmake to not expand variables of the form ${VAR}):

configure_file(mylib.pc.in mylib.pc @ONLY)
And finally, install generated file:

install(FILES ${CMAKE_BINARY_DIR}/mylib.pc DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)
You may also use cmake EXPORT feature. However, this feature is only compatible with cmake and I find it difficult to use.

Finally the entire CMakeLists.txt should looks like:

cmake_minimum_required(VERSION 3.9)
project(mylib VERSION 1.0.1 DESCRIPTION "mylib description")
include(GNUInstallDirs)
add_library(mylib SHARED src/mylib.c)
set_target_properties(mylib PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION 1
    PUBLIC_HEADER api/mylib.h)
configure_file(mylib.pc.in mylib.pc @ONLY)
target_include_directories(mylib PRIVATE .)
install(TARGETS mylib
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${CMAKE_BINARY_DIR}/mylib.pc
    DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)
