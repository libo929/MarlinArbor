########################################################
# cmake file for building MarlinArbor
# @author Jan Engels, DESY
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.2 FATAL_ERROR)
########################################################


# project name
PROJECT( MarlinArbor )


# project version
SET( MarlinArbor_VERSION_MAJOR 01 )
SET( MarlinArbor_VERSION_MINOR 04 )
SET( MarlinArbor_VERSION_PATCH 01 )


### CMAKE ###################################################################

# load default settings
INCLUDE( PandoraCMakeSettings )


### SOURCE ##################################################################

#include directories
INCLUDE_DIRECTORIES( ./include )
INSTALL( DIRECTORY ./include DESTINATION include FILES_MATCHING PATTERN "*.h" )

AUX_SOURCE_DIRECTORY( ./src/ MarlinArbor_SRCS )

IF( NOT CMAKE_CXX_FLAGS )
    SET( CMAKE_CXX_FLAGS "-std=c++11" )
    SET( PROCEED_WITHOUT_CXX_FLAGS ON )
ENDIF()

INCLUDE( CheckCXXCompilerFlag )

UNSET( COMPILER_SUPPORTS_CXX_FLAGS CACHE )
CHECK_CXX_COMPILER_FLAG( ${CMAKE_CXX_FLAGS} COMPILER_SUPPORTS_CXX_FLAGS )
IF( NOT COMPILER_SUPPORTS_CXX_FLAGS )
    MESSAGE( STATUS "The compiler ${CMAKE_CXX_COMPILER} does not support cxx flags ${CMAKE_CXX_FLAGS}" )
    IF( NOT PROCEED_WITHOUT_CXX_FLAGS )
        MESSAGE( FATAL_ERROR "CMake abort." )
    ENDIF()
    MESSAGE( STATUS "Proceed without cxx flags ${CMAKE_CXX_FLAGS}" )
    UNSET( CMAKE_CXX_FLAGS )
ENDIF()

SET( CMAKE_CXX_FLAGS "-Wall -Werror -ansi -pedantic -Wno-long-long -Wno-sign-compare -fno-strict-aliasing ${CMAKE_CXX_FLAGS}" )
LIST( REMOVE_ITEM CMAKE_CXX_FLAGS -Weffc++ )

MESSAGE( STATUS "CMAKE_CXX_FLAGS : ${CMAKE_CXX_FLAGS}" )

### DEPENDENCIES ############################################################

FIND_PACKAGE( Marlin 1.0 REQUIRED )
FIND_PACKAGE( MarlinUtil REQUIRED )
FIND_PACKAGE( PandoraSDK 02.00.00 REQUIRED )
FIND_PACKAGE( ArborContent 01.01.00 REQUIRED )

FOREACH( pkg Marlin MarlinUtil PandoraSDK ArborContent )
    IF( ${pkg}_FOUND )
        INCLUDE_DIRECTORIES( ${${pkg}_INCLUDE_DIRS} )
        LINK_LIBRARIES( ${${pkg}_LIBRARIES} )
        ADD_DEFINITIONS ( ${${pkg}_DEFINITIONS} )
    ENDIF()
ENDFOREACH()

IF( PANDORA_MONITORING )
    FIND_PACKAGE( ROOT 5.26.00 REQUIRED COMPONENTS Eve Geom RGL EG ) # minimum required ROOT version
    INCLUDE_DIRECTORIES( ${ROOT_INCLUDE_DIRS} )
    LINK_LIBRARIES( ${ROOT_LIBRARIES} ${ROOT_COMPONENT_LIBRARIES} )
      
    FIND_PACKAGE( PandoraMonitoring 01.00.00 REQUIRED )
    IF( PandoraMonitoring_FOUND )
        INCLUDE_DIRECTORIES( ${PandoraMonitoring_INCLUDE_DIRS} )
        LINK_LIBRARIES( ${PandoraMonitoring_LIBRARIES} )
        ADD_DEFINITIONS ( ${PandoraMonitoring_DEFINITIONS} )
        ADD_DEFINITIONS( "-DMONITORING" )
    ENDIF()
ENDIF()


### DOCUMENTATION ###########################################################

OPTION( INSTALL_DOC "Set to OFF to skip build/install Documentation" OFF )

IF( INSTALL_DOC AND EXISTS "${PROJECT_SOURCE_DIR}/doc/CMakeLists.txt" )
    ADD_SUBDIRECTORY( ./doc )
ELSE()
    MESSAGE( STATUS "INSTALL_DOC set to OFF" )
    SET( INSTALL_DOC OFF )
ENDIF()


### LIBRARY #################################################################

# add library
ADD_LIBRARY( MarlinArbor SHARED ${MarlinArbor_SRCS} )


### INSTALL #################################################################

# install library
INSTALL( TARGETS ${PROJECT_NAME} DESTINATION lib COMPONENT Runtime )

# install scripts files
INSTALL( DIRECTORY ./scripts DESTINATION scripts )

# display some variables and write them to cache
PANDORA_DISPLAY_STD_VARIABLES()
