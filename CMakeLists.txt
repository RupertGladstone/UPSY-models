cmake_minimum_required(VERSION 3.10)
project( UPSY_models LANGUAGES Fortran C)

set( CMAKE_Fortran_STANDARD 2018)

option(BUILD_UFEMISM "Build the UFEMISM model" OFF)
option(BUILD_LADDIE "Build the LADDIE model" OFF)
option(BUILD_LADDIE_LIBRARY "Also build LADDIE as a linkable static library (e.g. for coupling into FISOC)" OFF)

# When LADDIE is built as a library for external coupling, every object that ends up
# linked into the coupling executable (UPSY + UFEMISM + LADDIE) must be position-
# independent (-fPIC). Set this before any targets are created so it applies to all of them.
if(BUILD_LADDIE_LIBRARY)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

include( cmake/link_to_dependencies.cmake)
include( cmake/add_compiler_flags.cmake)
include( cmake/fortran_comment_sanitizer.cmake)

add_subdirectory(src/UPSY)

if(BUILD_UFEMISM)
    add_subdirectory(src/UFEMISM)
endif(BUILD_UFEMISM)

if(BUILD_LADDIE)
    add_subdirectory(src/LADDIE)
endif(BUILD_LADDIE)
