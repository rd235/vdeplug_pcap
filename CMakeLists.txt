cmake_minimum_required(VERSION 3.1)
project("vdeplug_pcap"
    DESCRIPTION "libvdeplug plugin module for pcap"
    HOMEPAGE_URL "https://github.com/rd235/vdeplug_pcap"
    VERSION 0.1.0
    LANGUAGES C)

include(GNUInstallDirs)
include(CheckIncludeFile)
include(CheckSymbolExists)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FORTIFY_SOURCE=2 -O2 -pedantic -Wall -Wextra")


set(CMAKE_REQUIRED_QUIET TRUE)
set(LIBS_REQUIRED vdeplug_mod pcap)
set(HEADERS_REQUIRED libvdeplug.h pcap/pcap.h)

foreach(THISLIB IN LISTS LIBS_REQUIRED)
  find_library(LIB${THISLIB}_OK ${THISLIB})
  if(NOT LIB${THISLIB}_OK)
    message(FATAL_ERROR "library lib${THISLIB} not found")
  endif()
endforeach(THISLIB)

foreach(HEADER IN LISTS HEADERS_REQUIRED)
  check_include_file(${HEADER} ${HEADER}_OK)
  if(NOT ${HEADER}_OK)
    message(FATAL_ERROR "header file ${HEADER} not found")
  endif()
endforeach(HEADER)

# check libpcap has pcap_create (i.e. version >= 1.0)
set(CMAKE_REQUIRED_LIBRARIES "pcap")
check_symbol_exists(pcap_create pcap/pcap.h HAVE_PCAP_CREATE)
if (NOT HAVE_PCAP_CREATE)
	message(FATAL_ERROR "function pcap_create not found")
endif()

add_definitions(-D_GNU_SOURCE)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_library(vdeplug_pcap SHARED libvdeplug_pcap.c)
target_link_libraries(vdeplug_pcap vdeplug_mod pcap)

install(TARGETS vdeplug_pcap DESTINATION ${CMAKE_INSTALL_LIBDIR}/vdeplug)

add_subdirectory(man)

add_custom_target(uninstall
  "${CMAKE_COMMAND}" -P "${PROJECT_SOURCE_DIR}/Uninstall.cmake")
