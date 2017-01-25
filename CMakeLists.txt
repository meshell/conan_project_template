# C++ Project template main cmake file using

# required minimum version of cmake
cmake_minimum_required(VERSION 3.2 FATAL_ERROR)

cmake_policy(SET CMP0048 NEW)

# project name
# TODO fill in your project name and version
project(Cpp_conan_project_template VERSION 0.1.0 LANGUAGES CXX C)

if(EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  set(CONAN_SYSTEM_INCLUDES ON)
  conan_basic_setup()
endif()


# project name used in doxygen docu
STRING(REGEX REPLACE "_" " " PROJECT_DOC_NAME ${PROJECT_NAME})

# project name used for installation
set(APPLICATION_NAME ${PROJECT_DOC_NAME})

# set default built type to Debug
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif(NOT CMAKE_BUILD_TYPE)

# Cmake test
enable_testing()

#
# Generic Compiler Flags
#

# Check for clang compiler
if(CMAKE_CXX_COMPILER MATCHES ".*clang")
  set(CMAKE_COMPILER_IS_CLANGCXX 1)
endif()

if(UNIX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Xlinker '--no-as-needed'")
endif(UNIX)

# set the path to the additional cmake modules
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)

find_package(Boost)
if(Boost_FOUND)
    add_subdirectory(src)
endif(Boost_FOUND)

# add a target to generate API documentation with Doxygen
set(DOC_DIR ${PROJECT_SOURCE_DIR}/doc CACHE PATH "Documentation directory")

file(MAKE_DIRECTORY
  ${DOC_DIR}
)

# Doxygen documentation
find_package(Doxygen)
if(DOXYGEN_FOUND)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doxygen/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
  add_custom_target(doc
    ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
    WORKING_DIRECTORY ${DOC_DIR}
    COMMENT "Generating API documentation with Doxygen." VERBATIM
  )

  install(DIRECTORY ${DOC_DIR}/html
    DESTINATION doc
    COMPONENT doc
  )
endif(DOXYGEN_FOUND)

install(FILES README.md LICENSE
  DESTINATION doc
  COMPONENT doc
)

# Targets for code anyalysis
# set report and documentation output paths
set(REPORT_DIR ${PROJECT_SOURCE_DIR}/reports CACHE PATH "Reports directory")

# create report and documentation output directories
file(MAKE_DIRECTORY
  ${REPORT_DIR}
)

find_program(CPPCHECK_EXE cppcheck)

if(CPPCHECK_EXE)
  add_custom_target(run_cppcheck
    ${CPPCHECK_EXE}  -iexternals --enable=style -f --std=c++11 -j2 --xml --suppress=*:externals/include/gtest/gtest.h --suppress=*:externals/include/gtest/internal/gtest-tuple.h --suppress=*:externals/include/gmock/gmock.h ./ 2> ${REPORT_DIR}/cppcheck.xml
    COMMENT "Run cppcheck and output report to ${REPORT_DIR}/cppcheck.xml" VERBATIM
  )
endif(CPPCHECK_EXE)

find_program(RATS_EXE rats)

if(RATS_EXE)
  add_custom_target(run_rats
    ${RATS_EXE}  --quiet --xml ./src > ${REPORT_DIR}/rats.xml
    COMMENT "Run cppcheck and output report to ${REPORT_DIR}/rats.xml" VERBATIM
  )
endif(RATS_EXE)

add_custom_target(clean-reports
  ${CMAKE_COMMAND} -E remove_directory ${REPORT_DIR}
  COMMENT "Delete all reports." VERBATIM
)

include(CMakeInstallation.cmake)
