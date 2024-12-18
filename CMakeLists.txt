cmake_minimum_required(VERSION 3.13)
project(stud-uuid VERSION 1.0.7 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libstud/uuid/version.hxx.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/libstud/uuid/version.hxx @ONLY)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

# Header not needed for build only for install
set(MAIN_HEADER libstud/uuid.hxx)

set(HEADERS
    libstud/uuid/uuid.hxx
    libstud/uuid/uuid-io.hxx
    libstud/uuid/export.hxx
    libstud/uuid/uuid.ixx
    ${CMAKE_CURRENT_BINARY_DIR}/libstud/uuid/version.hxx
)

set(SOURCES
    libstud/uuid/uuid.cxx
    libstud/uuid/uuid-io.cxx
)

add_library(stud-uuid STATIC ${sources})
add_library(stud::uuid ALIAS stud-uuid)

target_include_directories(stud-uuid
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:include>
)

set_target_properties(stud-uuid PROPERTIES
    OUTPUT_NAME stud-uuid
    VERSION ${PROJECT_VERSION}
    PUBLIC_HEADER "${HEADERS}"
)

if (LINUX)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-linux.cxx)
elseif (WIN32)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-windows.cxx)
    target_link_libraries(stud-uuid rpcrt4.lib)
elseif (APPLE)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-macos.cxx)
    target_link_libraries(stud-uuid "-framework CoreFoundation")
elseif (EMSCRIPTEN)
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-emscripten.cxx)
elseif (CMAKE_SYSTEM_NAME MATCHES "FreeBSD" OR CMAKE_SYSTEM_NAME MATCHES "NetBSD")
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-freebsd.cxx)
elseif (CMAKE_SYSTEM_NAME MATCHES "OpenBSD")
    target_sources(stud-uuid PRIVATE libstud/uuid/uuid-openbsd.cxx)
else()
    message(FATAL_ERROR "Unknown target platform.")
endif()


# Installation
include(GNUInstallDirs)

install(TARGETS stud-uuid
    EXPORT stud-uuid-targets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libstud/uuid
)

install(FILES ${MAIN_HEADER}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libstud
)

# Export targets
install(EXPORT stud-uuid-targets
    FILE stud-uuid-targets.cmake
    NAMESPACE stud::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/stud-uuid
)

# Create and install config file
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/stud-uuid-config-version.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/stud-uuid-config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/stud-uuid-config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/stud-uuid
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/stud-uuid-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/stud-uuid-config-version.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/stud-uuid
)
