# CMakeLists for the AppStream Project
project(appstream)
cmake_minimum_required(VERSION 3.2.0 FATAL_ERROR)

set(CMAKE_BUILD_TYPE "Debug")

set(AS_VERSION_MAJOR  "0")
set(AS_VERSION_MINOR  "8")
set(AS_VERSION_PATCH  "5")
set(AS_VERSION "${AS_VERSION_MAJOR}.${AS_VERSION_MINOR}.${AS_VERSION_PATCH}")
set(APPSTREAM_LIB_API_LEVEL "2")

# Don't allow in-tree building
if(${CMAKE_SOURCE_DIR} MATCHES ${CMAKE_BINARY_DIR})
      message(STATUS "Please do an out-of-tree build:")
      message(STATUS "rm -f CMakeCache.txt && mkdir build && cd build; cmake .. && make")
      message(FATAL_ERROR "In-tree-build detected!")
endif(${CMAKE_SOURCE_DIR} MATCHES ${CMAKE_BINARY_DIR})

#
# Options
#
option(DEBIAN_DEP11 "Enable DEP11 data provider" ON)
option(L18N "Enable localization" ON)
option(VAPI "Create and install a Vala API file" OFF)
option(QT "Build libappstream-qt" OFF)
option(TESTS "Enable Unit Tests" OFF)
option(MAINTAINER "Enable maintainer mode" OFF)
option(DOCUMENTATION "Enable target to build documentation" OFF)

if(DEBIAN_DEP11)
	add_definitions ("-DDEBIAN_DEP11")
endif()

#
# Default paths
#
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX
    "/usr" CACHE PATH "Default install prefix" FORCE)
endif(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)

# Special case for /etc and /var when prefix is /usr
if(${CMAKE_INSTALL_PREFIX} STREQUAL "/usr")
  set(CMAKE_INSTALL_SYSCONFDIR "/etc" CACHE PATH "read-only single-machine data (etc)")
  set(CMAKE_INSTALL_LOCALSTATEDIR "/var" CACHE PATH "modifiable single-machine data (var)")
endif(${CMAKE_INSTALL_PREFIX} STREQUAL "/usr")
include(GNUInstallDirs)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/data/cmake/)

message(STATUS "Compiling AppStream version ${AS_VERSION}")

add_custom_target(distclean
  "make clean"
)

#
# l18n
#
find_package (Gettext REQUIRED)

#
# Configure files
#
set (GETTEXT_PACKAGE "appstream")
set (VERSION "${AS_VERSION}")
configure_file(config.h.in ${CMAKE_BINARY_DIR}/config.h)

#
# Enable/disable testing
#
if(TESTS)
	enable_testing()
endif(TESTS)

#
# Custom C flags
#
set(MAINTAINER_CFLAGS "")
if(MAINTAINER)
	set(MAINTAINER_CFLAGS "-Werror -Wall -Wcast-align -Wno-uninitialized -Wempty-body -Wformat-security -Winit-self")
	if (CMAKE_COMPILER_IS_GNUCC)
		execute_process(COMMAND ${CMAKE_C_COMPILER} -dumpversion
                OUTPUT_VARIABLE GCC_VERSION)
		if (GCC_VERSION VERSION_GREATER 4.9 OR GCC_VERSION VERSION_EQUAL 4.9)
			set(MAINTAINER_CFLAGS ${MAINTAINER_CFLAGS} "-fdiagnostics-color=auto")
		endif()
	endif()
endif(MAINTAINER)
add_definitions(${MAINTAINER_CFLAGS})

# we do always enable this to never run into the missing-include-trap again
add_definitions("-Werror=implicit-function-declaration")

# enable C++11
set(CMAKE_CXX_STANDARD "11")

find_package(PkgConfig REQUIRED)

add_subdirectory(src)
add_subdirectory(tools)
add_subdirectory(data)
add_subdirectory(contrib)
add_subdirectory(po)
add_subdirectory(docs)
add_subdirectory(tests)
if(QT)
	add_subdirectory(qt)
endif()
