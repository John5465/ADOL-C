cmake_minimum_required(VERSION 3.19)

project(adol-c
  VERSION 2.7.3
  LANGUAGES C CXX
  DESCRIPTION "A Package for Automatic Differentiation of Algorithms Written in C/C++"
  HOMEPAGE_URL "https://github.com/coin-or/ADOL-C")

if (MSVC)
  message(STATUS "---- compiling with MSVC")
  cmake_policy(SET CMP0091 NEW)
  set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
  set(CMAKE_BUILD_TYPE DEBUG)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_FLAGS "/EHsc /source-charset:utf-8 /std:c++17")
endif()
set(CMAKE_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install)

option(ENABLE_SPARSE "Build with Colpack to enable sparse AD" ON)

add_library(adolc SHARED)
add_library(adolc::adolc ALIAS adolc)

target_compile_features(adolc PUBLIC cxx_std_17)

# Make the version of ADOL-C available as compile definitions
target_compile_definitions(adolc PRIVATE
  ADOLC_VERSION=${adol-c_VERSION_MAJOR}
  ADOLC_SUBVERSION=${adol-c_VERSION_MINOR}
  ADOLC_PATCHLEVEL=${adol-c_VERSION_PATCH})


# win specific flags
if (WIN32)
  target_compile_definitions(adolc PRIVATE     
      # handles __declspec
      ADOLC_DLL
      _WINDOWS
  )   
endif(WIN32) 


# Set the public include directory containing headers that will be installed
target_include_directories(adolc
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/ADOL-C/include>
    $<INSTALL_INTERFACE:include>)

target_include_directories(adolc
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/extra/ColPack/include>
)

# Set an include directory for the internally used library headers.
#
# This includes the files uni5_for.cpp, fo_rev.cpp, and ho_rev.cpp. Even though
# they end with .cpp, they are used like header files.  Together with some
# preprocessor trickery this is an old-fashioned way to do generic programming.
target_include_directories(adolc
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/ADOL-C/src>)


# handle the options
# ------------------

set(UINT_TYPE size_t)
set(REAL_TYPE double)

set(ADVBRANCH "#undef ADOLC_ADVANCED_BRANCHING")
set(ADTL_REFCNT "#undef USE_ADTL_REFCOUNTING")
#set(SPARSE_DRIVERS "#undef SPARSE_DRIVERS")

option(ENABLE_BOOST_POOL "Flag to activate boost-pool support" OFF)
if(ENABLE_BOOST_POOL)
  message(STATUS "Boost-pool support is enabled.")
  set(USE_BOOST_POOL "#define USE_BOOST_POOL 1")
else()
  message(STATUS "Boost-pool support is disabled.")
  set(USE_BOOST_POOL "#undef USE_BOOST_POOL")
endif()


target_compile_definitions(adolc PRIVATE
  $<$<BOOL:${ENABLE_SPARSE}>:SPARSE_DRIVERS=1>
  $<$<BOOL:${ENABLE_SPARSE}>:SPARSE=1>
  HAVE_LIBCOLPACK
)

find_library(COLPACK_LIB 
    NAMES ColPack_static.lib
    PATHS "${CMAKE_CURRENT_SOURCE_DIR}/extra/ColPack/lib"
    NO_DEFAULT_PATH
)
target_link_libraries(adolc PRIVATE ${COLPACK_LIB})

# include subdirectories for handling of includes and source files
# ----------------------------------------------------------------

add_subdirectory(ADOL-C)


# include the c interface of adolc
# ----------------------------------------------------------------
option(BUILD_INTERFACE OFF)
if(BUILD_INTERFACE)
    # set rpath for execution to the install location of adolc
    set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
    add_subdirectory(ADOL-C/c_interface)
    install(TARGETS ADOLCInterface EXPORT ADOLCInterfaceTargets)
    install(EXPORT ADOLCInterfaceTargets 
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/adolc)
endif()

# build the tests 
# ------------------------------------
option(BUILD_TESTS OFF)
if(BUILD_TESTS)
    set(ADOLC_INCLUDE_DIR "${CMAKE_BINARY_DIR}/ADOL-C/include")
    add_subdirectory(ADOL-C/boost-test)

    enable_testing()
    add_test(NAME boost-test-adolc
      COMMAND boost-test-adolc)
endif()

# build the adolc and tests with coverage
# ------------------------------------
option(BUILD_TESTS_WITH_COV OFF)
if(BUILD_TESTS_WITH_COV)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g --coverage")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g --coverage")

    set(ADOLC_INCLUDE_DIR "${CMAKE_BINARY_DIR}/ADOL-C/include")
    add_subdirectory(ADOL-C/boost-test)

    enable_testing()
    add_test(NAME boost-test-adolc
      COMMAND boost-test-adolc)
endif()


# export the targets
# ------------------

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

install(TARGETS adolc EXPORT adolcTargets)
install(EXPORT adolcTargets
  FILE adolc-targets.cmake
  NAMESPACE adolc::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/adolc)




