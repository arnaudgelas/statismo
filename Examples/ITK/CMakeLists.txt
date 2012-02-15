CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
PROJECT(ssm_itk)
SET(CMAKE_VERBOSE_MAKEFILE ON)
FIND_PATH(STATISMO_DIR STATISMO REQUIRED)
include(${STATISMO_DIR}/statismo-config.cmake)

FIND_PACKAGE(ITK REQUIRED)
include (${ITK_USE_FILE})

SET (ITK_STATISTICAL_MODEL_INCLUDE_DIR ${STATISMO_DIR}/ITKstatismo)

FILE(GLOB INCLUDE_FILES *.h)
FILE(GLOB SOURCE_FILES  *.txx )

include_directories(${STATISMO_DIR} ${STATISMO_DIR}/statismo_ITK  ${STATISMO_INCLUDE_DIR}  ${ITK_INCLUDE_DIRS}  ${STATISMO_DIR}/Representers/ITK)
link_directories(${ITK_LIBRARY_DIRS} ${HDF5_LIBRARY_DIR})

add_executable (ITKTestApp itkExample.cpp 
	${SOURCE_FILES}
	${INCLUDE_FILES}
) 
target_link_libraries (ITKTestApp ${ITK_LIBRARIES} ${HDF5_LIBRARIES})

add_executable (ITKFitting fitting.cpp 
	${SOURCE_FILES}
	${INCLUDE_FILES}
) 
target_link_libraries (ITKFitting ${ITK_LIBRARIES} ${HDF5_LIBRARIES})