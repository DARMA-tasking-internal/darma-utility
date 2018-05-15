
cmake_minimum_required(VERSION 3.4)

project (DarmaUtility
  VERSION 0.5
  LANGUAGES CXX
)

set(CMAKE_MODULE_PATH
  ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules
)

################################################################################
#------------------------------------------------------------------------------#
# darma_utility interface library                                              #
#------------------------------------------------------------------------------#

add_library(darma_utility INTERFACE)
add_library(darma_utility::darma_utility ALIAS darma_utility)

target_include_directories(darma_utility INTERFACE
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/source/include>
  $<INSTALL_INTERFACE:include>
)
target_compile_features(darma_utility INTERFACE cxx_std_14)

install(TARGETS darma_utility EXPORT darmaUtilityTargets
  INCLUDES DESTINATION include
)

install(EXPORT darmaUtilityTargets
  FILE darmaUtilityTargets.cmake
  NAMESPACE darma_utility::
  DESTINATION cmake
)

export(TARGETS darma_utility
  NAMESPACE darma_utility::
  FILE darmaUtilityTargets.cmake
)

################################################################################
# Dependencies
################################################################################

find_package(TinyMPL REQUIRED 2.0)

message(STATUS "Found TinyMPL: ${TinyMPL_DIR}")

target_link_libraries(darma_utility INTERFACE tinympl::tinympl)

################################################################################
# Compilation Checks, etc.
################################################################################

include (CheckCXXCompilerFlag)
include (CheckCXXSourceCompiles)

#-------------------------------------------------------------------------------
# Check for available attributes
set(CMAKE_REQUIRED_FLAGS_TMP ${CMAKE_REQUIRED_FLAGS})
set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS_TMP} -Werror=attributes -std=c++14 ${CMAKE_CXX_FLAGS}")

check_cxx_source_compiles("
[[ gnu::nonnull(1,2) ]]
extern void* my_memcpy(void *dest, const void *src);
int main() { }
"
  DARMA_HAVE_ATTRIBUTE_NONNULL
)

check_cxx_source_compiles("
[[ deprecated ]] void* my_memcpy(void *dest, const void *src);
int main() { }
"
  DARMA_HAVE_ATTRIBUTE_DEPRECATED
)

check_cxx_source_compiles("
[[ deprecated(\"reason\") ]]
void* my_memcpy(void *dest, const void *src);
int main() { }
"
  DARMA_HAVE_ATTRIBUTE_DEPRECATED_WITH_MESSAGE
)

check_cxx_source_compiles("
[[ gnu::always_inline ]] inline
int foo(int a) { return a + 5; }
int main() { }
"
  DARMA_HAVE_ATTRIBUTE_ALWAYS_INLINE
)

check_cxx_source_compiles("
[[ gnu::flatten ]]
int foo(int a) { return a + 5; }
int main() { }
"
  DARMA_HAVE_ATTRIBUTE_FLATTEN
)

set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS_TMP}")
# end of attribute checks
#-------------------------------------------------------------------------------

check_cxx_source_compiles("
#include <cxxabi.h>
int main() { }
"
  DARMA_HAVE_CXXABI_H
)

#-------------------------------------------------------------------------------
# Generate the configuration header

configure_file(
  source/include/darma/utility/config-generated.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/include/darma/utility/config-generated.h
)
install (
  FILES ${CMAKE_CURRENT_BINARY_DIR}/include/darma/utility/config-generated.h
  DESTINATION include/darma/utility
)
target_compile_definitions(darma_utility INTERFACE DARMA_CONFIG_GENERATED_H_AVAILABLE)

#-------------------------------------------------------------------------------

install(DIRECTORY source/include/darma/utility DESTINATION include/darma FILES_MATCHING PATTERN "*.h")

################################################################################

include(CMakePackageConfigHelpers)
configure_package_config_file(cmake/DarmaUtilityConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/DarmaUtilityConfig.cmake
  INSTALL_DESTINATION cmake
)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/DarmaUtilityConfigVersion.cmake
  COMPATIBILITY SameMajorVersion
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/DarmaUtilityConfig.cmake ${CMAKE_CURRENT_BINARY_DIR}/DarmaUtilityConfigVersion.cmake
    DESTINATION cmake
)

################################################################################

export(PACKAGE DarmaUtility)
