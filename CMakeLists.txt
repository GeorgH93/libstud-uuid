cmake_minimum_required(VERSION 3.24)
project(stud-uuid VERSION 1.0.7)
set(CMAKE_CXX_STANDARD 14)

option(STUD_UUID_STATIC "Compile Static library" ON)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libstud/uuid/version.hxx.in 
${CMAKE_CURRENT_BINARY_DIR}/libstud/uuid/version.hxx)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(sources ${sources}
        libstud/uuid/uuid.cxx libstud/uuid/uuid.hxx
        libstud/uuid/uuid-io.cxx libstud/uuid/uuid-io.hxx
        ${CMAKE_CURRENT_BINARY_DIR}/libstud/uuid/version.hxx)

if (STUD_UUID_STATIC)
	add_definitions(-DLIBSTUD_UUID_STATIC)
	message("[Compilation type: Static]")
	add_library(stud-uuid STATIC ${sources})

else()
	add_definitions(-DLIBSTUD_UUID_SHARED)
	message("[Compilation type: Shared]")
	add_library(stud-uuid SHARED ${sources})
endif(STUD_UUID_STATIC)

if(WIN32)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-windows.cxx)
    target_link_libraries(stud-uuid rpcrt4.lib)
elseif(APPLE)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-macos.cxx)
    target_link_libraries(stud-uuid "-framework CoreFoundation")
elseif(LINUX)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-linux.cxx)
    #target_link_libraries(stud-uuid ld)
elseif (CMAKE_SYSTEM_NAME MATCHES "FreeBSD" OR CMAKE_SYSTEM_NAME MATCHES "NetBSD")
    set(FREEBSD TRUE)
    set(BSD TRUE)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-freebsd.cxx)
elseif (CMAKE_SYSTEM_NAME MATCHES "OpenBSD")
    set(OPENBSD TRUE)
    set(BSD TRUE)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-openbsd.cxx)
elseif (EMSCRIPTEN)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-emscripten.cxx)
else()
    message(FATAL_ERROR "Target platform is not supported.")
endif()

target_include_directories(stud-uuid PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(stud-uuid PROPERTIES LINKER_LANGUAGE CXX)
set_target_properties(stud-uuid PROPERTIES VERSION ${PROJECT_VERSION})
