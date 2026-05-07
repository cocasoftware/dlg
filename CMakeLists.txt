# dlg CMakeLists.txt — COCA-managed build
# Upstream dlg (https://github.com/nyorain/dlg) ships only a meson.build.
# This CMakeLists is a hand-written, COCA-flavoured equivalent so the
# library participates in the same toolchain pipeline as every other
# Dependencies/ framework.

cmake_minimum_required(VERSION 3.20)
project(dlg
    VERSION 0.3.0
    DESCRIPTION "Pretty, lightweight C/C++ logging and assertion library"
    LANGUAGES C
)

# Single source, single public include root.
add_library(dlg STATIC src/dlg/dlg.c)
add_library(dlg::dlg ALIAS dlg)

# Static-library callers must see the DLG_STATIC define so the public
# headers don't apply __declspec(dllimport) on Windows.
target_compile_definitions(dlg PUBLIC DLG_STATIC)

target_include_directories(dlg
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

# Threading: dlg uses pthread on POSIX (and Windows native threads via the
# Win32 API on Windows; CMake's Threads package handles both portably).
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
target_link_libraries(dlg PUBLIC Threads::Threads)

target_compile_features(dlg PUBLIC c_std_11)

# Suppress upstream warnings (third-party code; clean of our concerns).
if(CMAKE_C_COMPILER_ID STREQUAL "MSVC")
    target_compile_options(dlg PRIVATE /w)
else()
    target_compile_options(dlg PRIVATE -w)
endif()

# Export for find_package(dlg) — same boilerplate as cityhash.
include(GNUInstallDirs)
install(TARGETS dlg EXPORT dlgTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(DIRECTORY include/dlg DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT dlgTargets
    FILE dlgConfig.cmake
    NAMESPACE dlg::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/dlg
)
