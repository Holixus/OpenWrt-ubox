cmake_minimum_required(VERSION 2.6)

PROJECT(ubox C)
ADD_DEFINITIONS(-Os -ggdb -Wall -Werror --std=gnu99 -Wmissing-declarations)

SET(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")

IF(APPLE)
  INCLUDE_DIRECTORIES(/opt/local/include)
  LINK_DIRECTORIES(/opt/local/lib)
ENDIF()

find_library(json NAMES json-c json)
find_library(ubox_lib NAMES ubox)
find_library(ubus_lib NAMES ubus)
find_library(uci_lib NAMES uci)
find_library(blobmsg_json_lib NAMES blobmsg_json)

IF(DEBUG)
  ADD_DEFINITIONS(-DDEBUG -g3)
ENDIF()

INCLUDE (CheckSymbolExists)
CHECK_SYMBOL_EXISTS(SYS_getrandom syscall.h getrandom)

ADD_EXECUTABLE(kmodloader kmodloader.c)
TARGET_LINK_LIBRARIES(kmodloader ${ubox_lib})

INSTALL(TARGETS kmodloader
	RUNTIME DESTINATION sbin
)

ADD_EXECUTABLE(lsbloader lsbloader.c)
TARGET_LINK_LIBRARIES(lsbloader ${ubox_lib} ${ubus_lib})

INSTALL(TARGETS lsbloader
	RUNTIME DESTINATION sbin
)

ADD_LIBRARY(validate SHARED validate/validate.c)
INSTALL(TARGETS validate
	LIBRARY DESTINATION lib
)

FIND_PATH(uci_include_dir uci.h)
INCLUDE_DIRECTORIES(${uci_include_dir})

ADD_EXECUTABLE(validate_data validate/cli.c)
TARGET_LINK_LIBRARIES(validate_data ${ubox_lib} ${uci_lib} validate)
INSTALL(TARGETS validate_data
	RUNTIME DESTINATION sbin
)

ADD_EXECUTABLE(logd log/logd.c log/syslog.c)
TARGET_LINK_LIBRARIES(logd ${ubox_lib} ${ubus_lib})
INSTALL(TARGETS logd
	RUNTIME DESTINATION sbin
)

ADD_EXECUTABLE(logread log/logread.c)
TARGET_LINK_LIBRARIES(logread ${ubox_lib} ${ubus_lib} ${json} ${blobmsg_json_lib})
INSTALL(TARGETS logread
	RUNTIME DESTINATION sbin
)
