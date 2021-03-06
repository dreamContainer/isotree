cmake_minimum_required (VERSION 3.8)
project (isotree VERSION 0.1.13)
set(CMAKE_BUILD_TYPE Release)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(SRC_FILES ${PROJECT_SOURCE_DIR}/src/crit.cpp
              ${PROJECT_SOURCE_DIR}/src/dealloc.cpp
              ${PROJECT_SOURCE_DIR}/src/dist.cpp
              ${PROJECT_SOURCE_DIR}/src/extended.cpp
              ${PROJECT_SOURCE_DIR}/src/fit_model.cpp
              ${PROJECT_SOURCE_DIR}/src/helpers_iforest.cpp
              ${PROJECT_SOURCE_DIR}/src/impute.cpp
              ${PROJECT_SOURCE_DIR}/src/isoforest.cpp
              ${PROJECT_SOURCE_DIR}/src/mult.cpp
              ${PROJECT_SOURCE_DIR}/src/predict.cpp
              ${PROJECT_SOURCE_DIR}/src/merge_models.cpp
              ${PROJECT_SOURCE_DIR}/src/serialize.cpp
              ${PROJECT_SOURCE_DIR}/src/utils.cpp)
add_library(isotree SHARED ${SRC_FILES})
target_include_directories(isotree PRIVATE ${PROJECT_SOURCE_DIR}/src)
set_target_properties(isotree PROPERTIES PUBLIC_HEADER ${PROJECT_SOURCE_DIR}/include/isotree.hpp)

add_compile_definitions(_USE_MERSENNE_TWISTER)

## https://stackoverflow.com/questions/12399422/how-to-set-linker-flags-for-openmp-in-cmakes-try-compile-function
find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
else()
	message(STATUS "OpenMP not found - will compile without multi-threading support")
endif()

# https://gist.github.com/OXPHOS/b7d9d31cd3841605df53effeb528c8e0
if (WIN32 OR MSVC)
  find_path(CEREAL_INCLUDE_DIRS cereal
    HINTS "$ENV{CMAKE_SOURCE_DIR}/include" "$ENV{CMAKE_BINARY_DIR}/cereal/include")
else()
  find_path(CEREAL_INCLUDE_DIRS cereal
    HINTS "$ENV{CMAKE_SOURCE_DIR}/include" "/usr/include" "/usr/local/include" "$ENV{CMAKE_BINARY_DIR}/cereal/include")
endif()
set(CEREAL_INCLUDE_DIRS ${CEREAL_INCLUDE_DIRS})
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(Cereal DEFAULT_MSG CEREAL_INCLUDE_DIRS)
mark_as_advanced(CEREAL_INCLUDE_DIRS)
if(CEREAL_FOUND)
  include_directories(${CEREAL_INCLUDE_DIRS})
  add_compile_definitions(_ENABLE_CEREAL)
else()
  message(STATUS "Cereal not found - will compile without serialization functionality")
endif(CEREAL_FOUND)

if (MSVC)
    add_compile_options(/O2)
else()
    add_compile_options(-O3 -march=native)
endif()

include(GNUInstallDirs)

install(TARGETS isotree
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

configure_file(isotree.pc.in isotree.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/isotree.pc DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)

# uninstall target
if(NOT TARGET uninstall)
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY)

    add_custom_target(uninstall
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif()
