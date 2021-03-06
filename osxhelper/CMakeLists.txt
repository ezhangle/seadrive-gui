CMAKE_MINIMUM_REQUIRED(VERSION 2.8.11)

PROJECT(seadrive-helper)
SET(PROJECT_VERSION "1.0.0")

IF (NOT (${CMAKE_BUILD_TYPE} MATCHES Release))
  SET(CMAKE_BUILD_TYPE Debug)
ENDIF()

MESSAGE(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

IF (NOT APPLE)
    MESSAGE(FATAL_ERROR "Support only on OS X")
ENDIF()

IF (NOT CMAKE_OSX_DEPLOYMENT_TARGET)
  SET(CMAKE_OSX_DEPLOYMENT_TARGET "10.7")
ENDIF()

## Build with warnings
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Wsign-compare -Wno-long-long -Wno-unused-parameter -fobjc-arc")

## Find the dependencies
FIND_LIBRARY(COCOA_LIBRARY Cocoa)
MARK_AS_ADVANCED (COCOA_LIBRARY)
FIND_LIBRARY(Sec_LIBRARY Security)
MARK_AS_ADVANCED (Sec_LIBRARY)
FIND_LIBRARY(SM_LIBRARY ServiceManagement)
MARK_AS_ADVANCED (SM_LIBRARY)
FIND_LIBRARY(IOKit_LIBRARY IOKit)
FIND_LIBRARY(MPMessagePack_LIBRARY MPMessagePack PATHS ${CMAKE_SOURCE_DIR}/../third_party)
SET(OSX_LIBS ${EXTRA_LIBS} ${COCOA_LIBRARY} ${Sec_LIBRARY} ${SM_LIBRARY} ${IOKit_LIBRARY} ${MPMessagePack_LIBRARY})

SET(CMAKE_EXE_LINKER_FLAGS "-fobjc-link-runtime ${CMAKE_EXE_LINKER_FLAGS}")

INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})
SET(HELPER_SRC
  helper-main.m
  helper-service.m
  helper-kext.m
  helper-log.m
  )

ADD_EXECUTABLE(seadrive-helper ${HELPER_SRC})

SET_TARGET_PROPERTIES(seadrive-helper PROPERTIES
  OUTPUT_NAME "com.seafile.seadrive.helper"
  # XCODE_ATTRIBUTE_MACOSX_DEPLOYMENT_TARGET "10.7"
  # XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC "YES"
  # XCODE_ATTRIBUTE_DEVELOPMENT_LANGUAGE "English"
  )


TARGET_LINK_LIBRARIES(seadrive-helper
  ${OSX_LIBS}
  # Embed the Info.plist and Launchd.plist into the helper tool executable
  "-sectcreate __TEXT __info_plist ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist -sectcreate __TEXT __launchd_plist ${CMAKE_CURRENT_SOURCE_DIR}/launchd.plist"
  # load all members from static objc libs like MPMessagePack because it has
  # objc class category
  "-ObjC"
)