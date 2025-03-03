cmake_minimum_required (VERSION 2.8)
project(Rocout)
#ENABLE_LANGUAGE( CXX Fortran )
#include(FortranCInterface)
#FortranCInterface_HEADER(${PROJECT_SOURCE_DIR}/include/FC.h MACRO_NAMESPACE "FC_")
#FortranCInterface_VERIFY(CXX QUIET)
#set (BUILD_STATIC FALSE CACHE BOOL "Build static COM library")
#set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single directory for all executables.")
#set (LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib CACHE PATH "Single directory for all libraries and archives.")
#mark_as_advanced (LIBRARY_OUTPUT_PATH EXECUTABLE_OUTPUT_PATH)
load_cache(../)
IF(NOT ENABLE_MPI)
  add_definitions( -DDUMMY_MPI )
ELSE()
  FIND_PACKAGE(MPI REQUIRED)
  add_definitions( -DMPICH_IGNORE_CXX_SEEK )
  INCLUDE_DIRECTORIES(${MPI_INCLUDE_PATH})
ENDIF()
IF(ENABLE_CGNS)
  add_definitions ( -DUSE_CGNS )
ENDIF()
IF(NOGLOB)
  add_definitions( -D_NO_GLOB_ )
ENDIF()
IF(USE_PTHREADS)
  add_definitions( -DUSE_PTHREADS )
ENDIF()

#INCLUDE(CTest)

#find_library(DL_LIB dl)
find_library(CGNS_LIB cgns)
find_path(CGNS_INC cgnslib.h)
#find_library(HDF4_LIB df)
#find_library(MFHDF_LIB mfhdf)
#find_library(ZLIB z)
find_path(HDF4_INC hdf.h HINTS /usr/include/hdf)
#find_library(JPEG_LIB jpeg)
#find_library(PTHREAD_LIB pthread)

IF(ENABLE_CGNS)
   set (ROCOUT_SRCS src/Rocout.C src/Rocout_hdf4.C src/write_parameter_file.C src/Rocout_cgns.C)
ELSE()
   set (ROCOUT_SRCS src/Rocout.C src/Rocout_hdf4.C src/write_parameter_file.C)
ENDIF()
set (TEST_SRCS test/outtest.C test/param_outtest.C)
set (UTIL_SRCS util/ghostbuster.C)
#set (RHDF4_SRCS src/HDF4.C)
#IF(USE_PTHREADS)
#set (RHDF4_SRCS ${RHDF4_SRCS} src/Sync.C)
#ENDIF()
set (ALL_SRCS "${ROCOUT_SRCS} ${TEST_SRCS} ${UTIL_SRCS}")
set(LIB_SRCS ${ROCOUT_SRCS})
#set (TEST_SRCS test/maptest.C)
set_source_files_properties(${ALL_SRCS} PROPERTIES COMPILE_FLAGS "-fPIC ${MPI_CXX_COMPILE_FLAGS}" )
#set(TEST_SRCS src/COMTest.C)

# rpath settings
SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

include_directories(include ${HDF4_INC})
IF(ENABLE_CGNS)
  include_directories(${CGNS_INC})
ENDIF()

IF(NOT BUILD_STATIC)
  add_library(Rocout SHARED ${ROCOUT_SRCS})
ELSE()
  add_library(Rocout STATIC ${ROCOUT_SRCS})
ENDIF()
IF(ENABLE_CGNS)
  target_link_libraries(Rocout ${CGNS_LIB})
ENDIF()
#target_link_libraries(Rocout Roccom)
#IF(USE_PTHREADS)
#  target_link_libraries(Rocout PTHREAD_LIB)
#ENDIF()
target_link_libraries(Rocout Rocin ${MPI_CXX_LIBRARIES})


# Test executables
add_executable(outtest test/outtest.C)
target_link_libraries(outtest Rocout ${MPI_CXX_LIBRARIES} ${HDF5_LIB})

IF(MPI_LINK_FLAGS)
  SET_TARGET_PROPERTIES(Rocout outtest PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
ENDIF()
#add_executable(inouttest test/inouttest.C)
#target_link_libraries(inouttest Rocout)
#add_executable(param_outtest test/param_outtest.C)
#target_link_libraries(param_outtest Rocout)

# Utilities
IF(ENABLE_CGNS)
  add_executable(ghostbuster util/ghostbuster.C)
  target_link_libraries(ghostbuster Rocout ${MPI_CXX_LIBRARIES} ${HDF5_LIB})
  IF(MPI_LINK_FLAGS)
    SET_TARGET_PROPERTIES(ghostbuster PROPERTIES LINK_FLAGS "${MPI_CXX_LINK_FLAGS}")
  ENDIF()
ENDIF()

#ADD_TEST(RunAllTests ${EXECUTABLE_OUTPUT_PATH}/testx testresults.txt)
#ADD_TEST(TestObject:ValidHandle ${TEST_RESULTS} TestObject:ValidHandle testresults.txt)
#ADD_TEST(TestObject:Exists ${TEST_RESULTS} TestObject:Exists testresults.txt)

INSTALL(TARGETS Rocout RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
IF(ENABLE_CGNS)
  INSTALL(TARGETS ghostbuster RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
ENDIF()
