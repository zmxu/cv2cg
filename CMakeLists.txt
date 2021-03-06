CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

# Avoid repeating the IF statement in the ENDIF and ELSE
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS true)

########### Project name ###########
SET(PACKAGE cv2cg)
PROJECT(${PACKAGE})
SET(VERSION_MAJOR "1")
SET(VERSION_MINOR "2")
SET(VERSION_PATCH "0")
SET(VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

########### Main global variables ###########
IF(NOT CMAKE_BUILD_TYPE)
 SET(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel." FORCE)
ENDIF()

IF(UNIX AND NOT WIN32)
 SET(CMAKE_CXX_FLAGS "-Wall -Wextra -Wno-unused-parameter")		# Enable lots of warning flags
ELSE()
	SET(CMAKE_CXX_FLAGS "/fp:precise /fp:except /bigobj /Zm1000 /EHsc /DWIN32")	# Refer to Stellarium. Flags used on purpose
ENDIF()

########### Find packages ###########
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/)
### OpenCV
FIND_PACKAGE(OpenCV REQUIRED)
SET(CV2CG_3RDPARTY_LINKEROPTION
	${CV2CG_3RDPARTY_LINKEROPTION}
	${OpenCV_LIBS}
)

### Eigen
#FIND_PACKAGE(Eigen3 REQUIRED)
#INCLUDE_DIRECTORIES(
#	${EIGEN3_INCLUDE_DIR}
#)

### FlyCap
OPTION(USE_FLYCAP "Use Point Grey FlyCapture2 library" OFF)
IF(USE_FLYCAP)
	FIND_PACKAGE(FlyCapture REQUIRED)
	INCLUDE_DIRECTORIES(
		${FLYCAPTURE_INCLUDE_PATH}
	)
	SET(CV2CG_3RDPARTY_LINKEROPTION
		${CV2CG_3RDPARTY_LINKEROPTION}
		${FLYCAPTURE_LIBRARY}
	)
ENDIF()

### LCM
OPTION(USE_LCM "Use LCM for inter-process communication" OFF)
IF(USE_LCM)
	FIND_PACKAGE(LCM REQUIRED)
	INCLUDE_DIRECTORIES(
		${LCM_INCLUDE_DIR}
		${LCM_INCLUDE_DIR}/WinSpecific/include
	)
	SET(CV2CG_3RDPARTY_LINKEROPTION
		${CV2CG_3RDPARTY_LINKEROPTION}
		${LCM_LIBRARY}
	)
ENDIF()

############################# create cv2cgConfig.h
CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/include/cv2cgConfig.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/include/cv2cgConfig.h")
INCLUDE_DIRECTORIES(
	"${CMAKE_CURRENT_BINARY_DIR}/include"
)

########### uninstall files ###############
CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)

ADD_CUSTOM_TARGET(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")

########### Macros ###############
include(${CMAKE_SOURCE_DIR}/cmake/AutoAppConfig.cmake)
include(${CMAKE_SOURCE_DIR}/cmake/AutoAddSubDir.cmake)
include(${CMAKE_SOURCE_DIR}/cmake/BuildLCMType.cmake)
  
########### Subdirectories ###############
ADD_SUBDIRECTORY(application)
