cmake_minimum_required(VERSION 3.15)

if(POLICY CMP0091)
    cmake_policy(SET CMP0091 NEW)
endif()

project(pkgconf VERSION 3.0.7 LANGUAGES C)

if(NOT WIN32)
    message(FATAL_ERROR "This CMake build is for the standalone native Windows executable. Use the upstream Meson build on other platforms.")
endif()

include(GNUInstallDirs)

if(MSVC)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endif()

set(PKGCONF_SOURCES
    libpkgconf/argvsplit.c
    libpkgconf/audit.c
    libpkgconf/bsdstubs.c
    libpkgconf/buffer.c
    libpkgconf/bufferset.c
    libpkgconf/bytecode.c
    libpkgconf/cache.c
    libpkgconf/client.c
    libpkgconf/dependency.c
    libpkgconf/fileio.c
    libpkgconf/fragment.c
    libpkgconf/license.c
    libpkgconf/output.c
    libpkgconf/parser.c
    libpkgconf/path.c
    libpkgconf/personality.c
    libpkgconf/pkg.c
    libpkgconf/queue.c
    libpkgconf/tuple.c
    libpkgconf/variable.c
    libpkgconf/version.c
)

file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/libpkgconf")
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/libpkgconf/config.h"
    @ONLY
)

add_library(libpkgconf STATIC ${PKGCONF_SOURCES})
target_include_directories(libpkgconf
    PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}"
        "${CMAKE_CURRENT_BINARY_DIR}"
)
target_compile_definitions(libpkgconf
    PRIVATE
        LIBPKGCONF_EXPORT
        PKGCONFIG_IS_STATIC
        _CRT_SECURE_NO_WARNINGS
)

set(VERSION "${PROJECT_VERSION}.0")
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/pkgconf.manifest.in"
    "${CMAKE_CURRENT_BINARY_DIR}/pkgconf.manifest"
    @ONLY
)
set(MANIFEST_FILE "${CMAKE_CURRENT_BINARY_DIR}/pkgconf.manifest")
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/pkgconf.rc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/pkgconf.rc"
    @ONLY
)

enable_language(RC)

add_executable(pkg-config
    cli/main.c
    cli/core.c
    cli/getopt_long.c
    cli/renderer-msvc.c
    "${CMAKE_CURRENT_BINARY_DIR}/pkgconf.rc"
)
target_include_directories(pkg-config
    PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}"
        "${CMAKE_CURRENT_BINARY_DIR}"
        "${CMAKE_CURRENT_SOURCE_DIR}/cli"
)
target_compile_definitions(pkg-config
    PRIVATE
        PKGCONFIG_IS_STATIC
        _CRT_SECURE_NO_WARNINGS
)
target_link_libraries(pkg-config PRIVATE libpkgconf advapi32)
set_target_properties(pkg-config PROPERTIES
    OUTPUT_NAME "pkg-config"
    LINK_FLAGS "/MANIFEST:NO"
)

install(TARGETS pkg-config RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
