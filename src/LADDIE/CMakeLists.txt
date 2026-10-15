# Collect LADDIE/UFEMISM source files
file( GLOB_RECURSE LADDIE_sources_raw  "${CMAKE_CURRENT_SOURCE_DIR}/*.f90")
list(REMOVE_ITEM LADDIE_sources_raw "${CMAKE_CURRENT_SOURCE_DIR}/main/LADDIE_program.f90")

file( GLOB_RECURSE UFEMISM_sources_raw "${PROJECT_SOURCE_DIR}/src/UFEMISM/*.f90")
file( GLOB UFEMISM_program_sources "${PROJECT_SOURCE_DIR}/src/UFEMISM/main/*_program.f90")

foreach(program_source IN LISTS UFEMISM_program_sources)
        list(REMOVE_ITEM UFEMISM_sources_raw "${program_source}")
endforeach()

sanitize_fortran_sources(shared_ufemism_laddie_sources_sanitized shared_ufemism_laddie ${LADDIE_sources_raw} ${UFEMISM_sources_raw})
sanitize_fortran_sources(LADDIE_program_source_sanitized LADDIE_program "${CMAKE_CURRENT_SOURCE_DIR}/main/LADDIE_program.f90")

if(BUILD_LADDIE_LIBRARY)

  # LADDIE static library: all LADDIE + UFEMISM sources except the program main.
  # Built as a library (rather than compiled straight into the executable) so that
  # external coupling code (e.g. the FISOC LADDIE ocean-model wrapper) can link against
  # LADDIE's initialise_forcing / initialise_laddie_model / run_laddie_model routines.
  # NB the UFEMISM objects are deliberately included: LADDIE reuses UFEMISM infrastructure
  # (reference geometry, ice-model types, masks, ice-shelf slopes, ocean setup), even though
  # it never runs UFEMISM ice dynamics. Mirrors the UPSY_static_library pattern. Position-
  # independent code is forced globally by the top-level CMakeLists when this option is on.
  add_library( LADDIE_library STATIC ${shared_ufemism_laddie_sources_sanitized})
  target_link_libraries( LADDIE_library PUBLIC UPSY_static_library)
  link_to_dependencies( LADDIE_library)
  add_compiler_flags( LADDIE_library)

  # Expose the library's module (.mod) directory to consumers (the program below, and
  # eventually the FISOC wrapper), and emit LADDIE's .mod files there.
  if(EXISTS ${PROJECT_SOURCE_DIR}/include)
    target_include_directories( LADDIE_library PUBLIC ${PROJECT_SOURCE_DIR}/include)
  endif()
  target_include_directories( LADDIE_library PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/mod_files)
  set_property( TARGET LADDIE_library PROPERTY Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/mod_files)

  # LADDIE program: just the main program unit, linked against the LADDIE library.
  add_executable( LADDIE_program ${LADDIE_program_source_sanitized})
  target_link_libraries( LADDIE_program PUBLIC LADDIE_library)
  link_to_dependencies( LADDIE_program)
  add_compiler_flags( LADDIE_program)

else()

  # Default standalone build: compile all LADDIE + UFEMISM sources straight into the
  # executable. No separate library, no forced position-independent code.
  add_executable( LADDIE_program ${shared_ufemism_laddie_sources_sanitized} ${LADDIE_program_source_sanitized})
  target_link_libraries( LADDIE_program PUBLIC UPSY_static_library)
  link_to_dependencies( LADDIE_program)
  add_compiler_flags( LADDIE_program)

endif()
