cmake_minimum_required(VERSION 3.22)
set(SC_MODULE geometry)
project(simply-cpp-${SC_MODULE} HOMEPAGE_URL "https://github.com/roelofrossouw/simply-cpp-geometry" LANGUAGES CXX)
include(cmake/sc_bootstrap.cmake)

option(SC_GEOMETRY_BUILD_TESTS "Build simply-cpp-geometry tests" ${PROJECT_IS_TOP_LEVEL})

get_sc_version()
message(STATUS "SimplyCPP ${SC_MODULE} Version: ${SC_VERSION}")

# sc-core for the helpers, the tests and the examples; the library itself uses only the standard
# library (it moved out of sc-core in 4.0.0).
find_or_install_package(sc-core simply-cpp-dev simply-cpp 3.0.0) # TODO-CYCLE: 4.0.0

set(SOURCE_OBJECTS)
set(SOURCE_LIBRARIES)
set(SOURCE_LINK_LIBRARIES)

add_sc_object(size)
add_sc_object(rect)
add_sc_object(path)
add_sc_object(transform)
add_sc_object(percent)

add_sc_libraries(DESCRIPTION "simply-cpp geometry: points, sizes, shapes (rect, polygon, circle, line, path), affine transforms, distances in real units, percentages, matrices and dual numbers")
target_compile_features(sc-geometry PUBLIC cxx_std_20)
target_compile_features(sc-geometry-shared PUBLIC cxx_std_20)

install_sc_module(PATH_VARS CMAKE_INSTALL_INCLUDEDIR)
install(FILES cmake/SimplyCppFunctions.cmake cmake/sc_bootstrap.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/sc-geometry
        COMPONENT development)

if (SC_GEOMETRY_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests sc-geometry-test)
endif ()

add_sc_example(sc-geometry-demo LINK_LIBRARIES sc::sc-core sc::sc-geometry-shared)
add_sc_example(sc-geometry-numbers LINK_LIBRARIES sc::sc-core sc::sc-geometry-shared)

# The headers were sc-core's until 4.0.0 (as <sc/core/x.h>); they now live in <sc/geometry/x.h>, so no
# files are taken over from simply-cpp-dev.
package_sc_module()
