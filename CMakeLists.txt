cmake_minimum_required(VERSION 2.8.12)

# Define a single cmake project
project(preprocess)

#Set for FindICU.cmake
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})
set (CMAKE_CXX_STANDARD 11)

option(FORCE_STATIC "Build static executables" OFF)
if (FORCE_STATIC)
  #presumably overkill, is there a better way?
  #http://cmake.3232098.n2.nabble.com/Howto-compile-static-executable-td5580269.html
  set(Boost_USE_STATIC_LIBS ON)
  set_property(GLOBAL PROPERTY LINK_SEARCH_START_STATIC ON)
  set_property(GLOBAL PROPERTY LINK_SEARCH_END_STATIC ON)
  set(BUILD_SHARED_LIBRARIES OFF)
  if (MSVC)
	set(flag_vars
	  CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
	  CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
	  CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
	  CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)
	  foreach(flag_var ${flag_vars})
	    if(${flag_var} MATCHES "/MD")
	      string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
	    endif(${flag_var} MATCHES "/MD")
	  endforeach(flag_var)
  else (MSVC)
    if (NOT CMAKE_C_COMPILER_ID MATCHES ".*Clang")
      set(CMAKE_EXE_LINKER_FLAGS "-static-libgcc -static-libstdc++ -static")
    endif ()
  set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
  endif ()
  #Annoyingly the exectuables say "File not found" unless these are set
  set(CMAKE_EXE_LINK_DYNAMIC_C_FLAGS)
  set(CMAKE_EXE_LINK_DYNAMIC_CXX_FLAGS)
  set(CMAKE_SHARED_LIBRARY_C_FLAGS)
  set(CMAKE_SHARED_LIBRARY_CXX_FLAGS)
  set(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS)
  set(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS)
endif ()

# Compile all executables into bin/
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

# Compile all libraries into lib/
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# Tell cmake that we want unit tests to be compiled
include(CTest)
enable_testing()

if(MSVC)
  set(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} /w34716")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /w34716")
endif()

# We need boost
find_package(Boost 1.41.0 REQUIRED COMPONENTS
  program_options
  system
  thread
  unit_test_framework
)

find_package(ICU REQUIRED COMPONENTS i18n uc data io)

# Define where include files live
include_directories(
  ${PROJECT_SOURCE_DIR}
  ${Boost_INCLUDE_DIRS}
  ${ICU_INCLUDE_DIRS}
)

# Process subdirectories
add_subdirectory(util)
add_subdirectory(preprocess)
add_subdirectory(moses)

