################################################################################
# Part of CMake configuration for GEOS
#
# Copyright (C) 2018 Mateusz Loskot <mateusz@loskot.net>
# Copyright (C) 2019 Daniel Baston <dbaston@gmail.com>
#
# This is free software; you can redistribute and/or modify it under
# the terms of the GNU Lesser General Public Licence as published
# by the Free Software Foundation.
# See the COPYING file for more information.
################################################################################
file(GLOB_RECURSE _sources ${CMAKE_CURRENT_LIST_DIR}/*.cpp CONFIGURE_DEPEND)
add_executable(test_geos_unit ${_sources})
unset(_sources)

find_package(Threads)

target_link_libraries(test_geos_unit PRIVATE geos geos_c Threads::Threads)
target_include_directories(test_geos_unit
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>)

file(GLOB_RECURSE _testfiles ${CMAKE_CURRENT_LIST_DIR}/**/*Test.cpp CONFIGURE_DEPEND)
foreach(_testfile ${_testfiles})
    string(REPLACE ${CMAKE_CURRENT_LIST_DIR}/ "" _testname ${_testfile})
    string(REPLACE "Test.cpp" "" _testname ${_testname})
    string(REPLACE "/" "-" _cmake_testname  ${_testname})
    string(REPLACE "/" "::" _testname ${_testname})
    if (NOT ${_testname} MATCHES "^capi::")
      string(CONCAT _testname "geos::" ${_testname})
    endif()
    add_test(NAME unit-${_cmake_testname} COMMAND test_geos_unit ${_testname})
endforeach()

# Disable tests that segfault on Windows MSVC Release builds with VS2026
if(MSVC AND MSVC_VERSION GREATER_EQUAL 1950)
  set_tests_properties(unit-capi-GEOSDifferencePrec PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-capi-GEOSGeom_setPrecision PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-capi-GEOSIntersectionPrec PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-capi-GEOSSymDifferencePrec PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-noding-snapround-SnapRoundingNoder PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-operation-buffer-BufferOp PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-operation-overlayng-OverlayNGMixedPoints PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-operation-overlayng-OverlayNGStrictMode PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-operation-overlayng-OverlayNG PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-operation-overlayng-PrecisionReducer PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-operation-overlayng-UnaryUnionNG PROPERTIES DISABLED $<CONFIG:Release>)
  set_tests_properties(unit-precision-GeometryPrecisionReducer PROPERTIES DISABLED $<CONFIG:Release>)
endif()

# Run all the unit tests in one go, for faster memory checking
# under valgrind. Restrict to one configuration so it is only
# run with 'ctest -C Valgrind'
# Exemplar commandline:
#
# ctest --output-on-failure \
#   --overwrite MemoryCheckCommandOptions="--leak-check=full --error-exitcode=100" \
#   -R all-unit-tests -C Valgrind -T memcheck
#
add_test(NAME all-unit-tests
    COMMAND test_geos_unit
    CONFIGURATIONS Valgrind
    )

unset(_cmake_testname)
unset(_testfile)
unset(_testfiles)
unset(_testname)

