cmake_minimum_required(VERSION 2.8.5)
project(statismo)

enable_testing()

set(STATISMO_VERSION 0.8)
set(STATISMO_MINOR_VERSION 1)


# Setup build locations.
if(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
endif()

#
# Compile and Install HDF5
#
include(ExternalProject)
option(HDF5_USE_EXTERNAL "Use External Library Building for HDF5" OFF)

if(NOT HDF5_USE_EXTERNAL)
  # it seems that on linux only the shared libraries work, while on windows static seems to work fine
  if(WIN32)
	  set(HDF5_BUILD_SHARED OFF)
	  set(HDF5_LIBRARIES debug hdf5d debug hdf5_cppd optimized hdf5; optimized hdf5_cpp)
	  set(HDF5_BUILD_TYPE "RELEASE" CACHE STRING "Build type (RELEASE DEBUG) of the hdf5 library" )
  else(WIN32)
	  set(HDF5_BUILD_SHARED ON)
	  set(HDF5_LIBRARIES debug hdf5 debug hdf5_cpp optimized hdf5; optimized hdf5_cpp)
	  set(HDF5_BUILD_TYPE "RELEASE")
  endif(WIN32)

	ExternalProject_add(HDF5
	  SOURCE_DIR ${CMAKE_BINARY_DIR}/HDF5
	  BINARY_DIR ${CMAKE_BINARY_DIR}/HDF5-build
	  URL "${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/hdf5-1.8.7.tar.gz"
	  UPDATE_COMMAND ""
	  CMAKE_ARGS
	  -DCMAKE_BUILD_TYPE:STRING=${HDF5_BUILD_TYPE}
	  -DHDF5_ENABLE_Z_LIB_SUPPORT:BOOL=OFF
	  -DHDF5_BUILD_CPP_LIB:BOOL=ON
	  -DBUILD_SHARED_LIBS:BOOL=${HDF5_BUILD_SHARED}
	  -DHDF5_BUILD_TOOLS:BOOL=OFF
	  -DCMAKE_INSTALL_PREFIX:PATH=${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/
	  INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/
	)
	set(HDF5_INCLUDE_DIR ${CMAKE_BINARY_DIR}/3rdParty/HDF5/include)
	set(HDF5_INCLUDE_DIR_CPP ${HDF5_INCLUDE_DIR}/cpp)
	set(HDF5_LIBRARY_DIR ${CMAKE_BINARY_DIR}/3rdParty/HDF5/lib)

  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/include/ DESTINATION include/hdf5 FILES_MATCHING PATTERN "*.h")
  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/include/cpp/ DESTINATION include/hdf5/cpp FILES_MATCHING PATTERN "*.h")
  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/lib/ DESTINATION lib/hdf5 FILES_MATCHING PATTERN "*hdf5*")
  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/bin/ DESTINATION bin FILES_MATCHING PATTERN "*hdf5*")

  set(STATISMO_LIBRARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/HDF5/lib/)

else(NOT HDF5_USE_EXTERNAL)

 	find_path(HDF5_INCLUDE_DIR hdf5.h)
 	find_path(HDF5_INCLUDE_DIR_CPP H5Cpp.h)
 	find_library(HDF5_LIBRARY hdf5)
 	find_library(HDF5_CPP_LIBRARY hdf5_cpp)
	get_filename_component(HDF5_LIBRARY_DIR ${HDF5_LIBRARY} PATH)
	set(HDF5_LIBRARIES ${HDF5_LIBRARY} ${HDF5_CPP_LIBRARY})
	set(STATISMO_LIBRARY_DIR ${HDF5_LIBRARY_DIR})

endif(NOT HDF5_USE_EXTERNAL)

# rpath settings
set(CMAKE_SKIP_BUILD_RPATH  FALSE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib;${STATISMO_LIBRARY_DIR}")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

#
#optional examples and wrapping
option(BUILD_ITK_EXAMPLES "Build the itk examples " OFF)
option(BUILD_VTK_EXAMPLES "Build the vtk examples " OFF)
option(BUILD_PYTHON_WRAPPING_ITK "Build python wrapping for the itk representers using wrapitk (experimental)" OFF)
option(BUILD_PYTHON_WRAPPING_VTK "Build python wrapping for the VTK representers" OFF)

mark_as_advanced(BUILD_PYTHON_WRAPPING_ITK)

#
#optional tests
#
option(BUILD_STATISMO_TESTS "Build tests" OFF)
option(BUILD_STATISMO_TESTS_PYTHON "Build Unit tests written in python (requires python wrapping)")
option(BUILD_REPRESENTER_TESTS "Build representer tests (requires ITK and VTK)" OFF)
mark_as_advanced(BUILD_REPRESENTER_TESTS)

option(HAS_CXX11_ASYNC "Statismo can make use of multi-threading in case the compiler supports std::async of the C++11 standard " OFF)
if(HAS_CXX11_ASYNC )
  add_definitions( -DHAS_CXX11_ASYNC )
endif(HAS_CXX11_ASYNC)


#
# Install boost and eigen, by just copying them from the 3rdParty directory
#
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/boost DESTINATION include/)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/Eigen DESTINATION include/)

#
# Install the statismo and statismo itk files (these are header only, so no compilation needed)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/statismo DESTINATION include)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/statismo_ITK DESTINATION include)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Representers DESTINATION include)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/statismo-config.cmake.in  ${CMAKE_CURRENT_BINARY_DIR}/statismo-config.cmake @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/statismo-config.cmake DESTINATION lib/cmake/statismo-${STATISMO_VERSION}${STATISMO_MINOR_VERSION})

#
# copy the data
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data DESTINATION share)

set(STATISMO_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR})

if(BUILD_ITK_EXAMPLES)
  add_subdirectory(Examples/ITK)
endif()

if(BUILD_VTK_EXAMPLES)
  add_subdirectory(Examples/VTK)
endif()

if(BUILD_STATISMO_TESTS)
  add_subdirectory(Tests)
endif()

if(BUILD_REPRESENTER_TESTS)
add_subdirectory(Tests/representerTests)
endif()

if(BUILD_STATISMO_TESTS_PYTHON)
  set(BUILD_PYTHON_WRAPPING_VTK ON)
  add_custom_target(unit_test  ${CMAKE_CURRENT_SOURCE_DIR}/Tests/statismoTests/runPythonTests.sh ${CMAKE_CURRENT_BINARY_DIR}/Wrapping/SWIG ${CMAKE_CURRENT_SOURCE_DIR}/Tests/statismoTests/)
endif()


if(BUILD_PYTHON_WRAPPING_VTK)
  add_subdirectory(Wrapping/SWIG)
endif()

if(BUILD_PYTHON_WRAPPING_ITK)
  add_subdirectory(Wrapping/ITK)
endif()
