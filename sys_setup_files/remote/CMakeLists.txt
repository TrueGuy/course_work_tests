# The name of our project is "MPDROOT".  CMakeLists files in this project can
# refer to the root source directory of the project as ${MPDROOT_SOURCE_DIR} 
# or as ${CMAKE_SOURCE_DIR} and to the root binary directory of the project as 
# ${MPDROOT_BINARY_DIR} or ${CMAKE_BINARY_DIR}.
# This difference is important for the base classes which are in FAIRROOT, 
# PANDAROOT, and MPDROOT.

# Check if cmake has the required version
CMAKE_MINIMUM_REQUIRED(VERSION  2.6.2 FATAL_ERROR)

# Set name of our project to "MPDROOT". Has to be done
# after check of cmake version                        
project(MPDROOT)

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ 
# is checked
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/cuda" ${CMAKE_MODULE_PATH})

# Load some basic macros which are needed later on
include(FairMacros)
include(WriteConfigFile)
include(CTest)
include(CheckCompiler)

# Set the build type. Possibilities are None, Debug, Release, 
# RelWithDebInfo and MinSizeRel 
# SET(CMAKE_BUILD_TYPE Debug)
If(NOT CMAKE_BUILD_TYPE)
       Message("Set BuildType DEBUG")
       set(CMAKE_BUILD_TYPE Debug)
EndIf(NOT CMAKE_BUILD_TYPE)

#Check the compiler and set the compile and link flags
Check_Compiler()

set(LIBRARY_OUTPUT_PATH "${CMAKE_BINARY_DIR}/lib")
set(EXECUTABLE_OUTPUT_PATH "${CMAKE_BINARY_DIR}/bin")

Option(USE_PATH_INFO "Information from PATH and LD_LIBRARY_PATH are
used." OFF)
If(USE_PATH_INFO)
  Set(PATH $ENV{PATH})
  If (APPLE)
    Set(LD_LIBRARY_PATH $ENV{DYLD_LIBRARY_PATH})
  Else (APPLE)
    Set(LD_LIBRARY_PATH $ENV{LD_LIBRARY_PATH})
  EndIf (APPLE)
EndIf(USE_PATH_INFO)

# Check if the user wants to build the project in the source
# directory
CHECK_OUT_OF_SOURCE_BUILD()

# Check if we are on an UNIX system. If not stop with an error
# message
IF(NOT UNIX)
  MESSAGE(FATAL_ERROR "You're not on an UNIX system. The project was up to now only tested on UNIX systems, so we break here. If you want to go on please edit the CMakeLists.txt in the source directory.")
ENDIF(NOT UNIX)  

#Check if necessary environment variables are set
#If not stop execution
IF(NOT DEFINED ENV{SIMPATH})
  MESSAGE(FATAL_ERROR "You did not define the environment variable SIMPATH which is nedded to find the external packages. Please set this variable and execute cmake again.")
ENDIF(NOT DEFINED ENV{SIMPATH})
STRING(REGEX MATCHALL "[^:]+" PATH $ENV{PATH})
SET(SIMPATH $ENV{SIMPATH})

# searches for needed packages
# REQUIRED means that cmake will stop if this packages are not found
# For example the framework can run without GEANT4, but ROOT is
# mandatory
find_package(ROOT REQUIRED)
find_package(PLUTO REQUIRED)
find_package(GENERATORS REQUIRED)
find_package(GEANT3 REQUIRED)
find_package(GEANT4)
find_package(GEANT4DATA)
find_package(GEANT4VMC)
find_package(CLHEP)
find_package(RuleChecker)
#find_package(Oracle)
#find_package(CUDA REQUIRED)

  Set(Boost_NO_SYSTEM_PATHS TRUE)
  Set(Boost_NO_BOOST_CMAKE TRUE)
#  set(Boost_DEBUG TRUE)
  If(${ROOT_LIBRARY_DIR} MATCHES /lib/root)
    set(BOOST_ROOT ${SIMPATH})
    set(GSL_DIR ${SIMPATH})
  Else(${ROOT_LIBRARY_DIR} MATCHES /lib/root)
    set(BOOST_ROOT ${SIMPATH}/basics/boost)
    set(GSL_DIR ${SIMPATH}/basics/gsl)
  EndIf(${ROOT_LIBRARY_DIR} MATCHES /lib/root)
  Message("-- Looking for Boost ...")
  # If an older version of boost is found both of the variables below are
  # cached and in a second cmake run, a good boost version is found even
  # if the version is to old.
  # To overcome this problem both variables are cleared before checking
  # for boost.
  Unset(Boost_INCLUDE_DIR CACHE)
  Unset(Boost_LIBRARY_DIRS CACHE)
  find_package(Boost 1.41)
  If (Boost_FOUND)
    Set(Boost_Avail 1)
  Else (Boost_FOUND)
    Set(Boost_Avail 0)
  EndIf (Boost_FOUND)

# Set the library version in the main CMakeLists.txt
SET(FAIRROOT_MAJOR_VERSION 0)
SET(FAIRROOT_MINOR_VERSION 0)
SET(FAIRROOT_PATCH_VERSION 0)
SET(FAIRROOT_VERSION "${FAIRROOT_MAJOR_VERSION}.${FAIRROOT_MINOR_VERSION}.${FAIRROOT_PATCH_VERSION}")
SET(FAIRROOT_LIBRARY_PROPERTIES ${FAIRROOT_LIBRARY_PROPERTIES}
    VERSION "${FAIRROOT_VERSION}"
    SOVERSION "${FAIRROOT_MAJOR_VERSION}"
    SUFFIX ".so"
)

SET(FAIRLIBDIR ${CMAKE_BINARY_DIR}/lib)  
SET(LD_LIBRARY_PATH ${FAIRLIBDIR} ${LD_LIBRARY_PATH})
SET(PATH ${EXECUTABLE_OUTPUT_PATH} ${PATH})

# Recurse into the given subdirectories.  This does not actually
# cause another cmake executable to run.  The same process will walk through
# the project's entire directory structure.
add_subdirectory (fairtools)
add_subdirectory (base)
add_subdirectory (geobase)
add_subdirectory (parbase) 
add_subdirectory (generators)
add_subdirectory (geane)
add_subdirectory (trackbase)
#add_subdirectory (cuda)
add_subdirectory (eventdisplay)
add_subdirectory (MbsAPI)
add_subdirectory (mpdbase)
add_subdirectory (passive)
add_subdirectory (mcstack)
add_subdirectory (mpdfield)
add_subdirectory (mpddata)
add_subdirectory (mpdgenerators)
add_subdirectory (lhetrack)
add_subdirectory (kalman)
add_subdirectory (macro)
add_subdirectory (macro/mpd_scheduler)
add_subdirectory (tpc)
add_subdirectory (tof)
add_subdirectory (zdc)
add_subdirectory (emc)
add_subdirectory (ffd)
#add_subdirectory (bbc)
add_subdirectory (strawendcap)
#add_subdirectory (etof)
#add_subdirectory (sft)
#add_subdirectory (stt)
#add_subdirectory (sts)
#add_subdirectory (cpc)
#add_subdirectory (fsa)
#add_subdirectory (ndet)
#add_subdirectory (tgem)
#add_subdirectory (dch)
add_subdirectory (shield_pack)
add_subdirectory (strawECT)
add_subdirectory (physics/femto)

Option(BUILD_DOXYGEN "Build Doxygen" OFF)
if(BUILD_DOXYGEN)
  MESSAGE(STATUS "*** Building the Doxygen documentaion ***")
  ADD_SUBDIRECTORY(doxygen)
endif(BUILD_DOXYGEN)

if(RULE_CHECKER_FOUND)

  ADD_CUSTOM_TARGET(RULES 
     COMMAND ${RULE_CHECKER_SCRIPT1} ${CMAKE_BINARY_DIR} viol > violations.html
     DEPENDS $ENV{ALL_RULES})

endif(RULE_CHECKER_FOUND)

WRITE_CONFIG_FILE(config.sh)
