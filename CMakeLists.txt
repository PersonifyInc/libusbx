cmake_minimum_required (VERSION 2.8)

project(libusbx)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/.. ${CMAKE_MODULE_PATH})

include_directories(${CMAKE_CURRENT_LIST_DIR}/config/mac)
include_directories(${CMAKE_CURRENT_LIST_DIR}/libusb)

set (LIBUSB_SRC
    ${CMAKE_CURRENT_LIST_DIR}/libusb/core.c
    ${CMAKE_CURRENT_LIST_DIR}/libusb/descriptor.c
    ${CMAKE_CURRENT_LIST_DIR}/libusb/hotplug.c
    ${CMAKE_CURRENT_LIST_DIR}/libusb/hotplug.h
    ${CMAKE_CURRENT_LIST_DIR}/libusb/io.c
    ${CMAKE_CURRENT_LIST_DIR}/libusb/libusb.h
    ${CMAKE_CURRENT_LIST_DIR}/libusb/libusbi.h
    ${CMAKE_CURRENT_LIST_DIR}/libusb/strerror.c
    ${CMAKE_CURRENT_LIST_DIR}/libusb/sync.c
    ${CMAKE_CURRENT_LIST_DIR}/libusb/version_nano.h
    ${CMAKE_CURRENT_LIST_DIR}/libusb/version.h
    )

if(APPLE)
    set (OS_SRC
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/darwin_usb.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/darwin_usb.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/threads_posix.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/threads_posix.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/poll_posix.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/poll_posix.h
    )
elseif(UNIX)
    set (OS_SRC
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/linux_netlink.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/linux_usbfs.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/linux_usbfs.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/linux_udev.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/threads_posix.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/threads_posix.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/poll_posix.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/poll_posix.h
    )
elseif(WIN32)
    set (OS_SRC
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/windows_usb.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/windows_usb.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/windows_common.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/threads_windows.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/threads_windows.h
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/poll_windows.c
        ${CMAKE_CURRENT_LIST_DIR}/libusb/os/poll_windows.h
    )
endif()

add_library(usbx SHARED ${LIBUSB_SRC} ${OS_SRC})

if(APPLE)
# This macro helps link OS X frameworks
macro(ADD_FRAMEWORK fwname appname)
    find_library(FRAMEWORK_${fwname}
    NAMES ${fwname}
    PATHS ${CMAKE_OSX_SYSROOT}/System/Library
    PATH_SUFFIXES Frameworks
    NO_DEFAULT_PATH)
    if( ${FRAMEWORK_${fwname}} STREQUAL FRAMEWORK_${fwname}-NOTFOUND)
        MESSAGE(ERROR ": Framework ${fwname} not found")
    else()
        TARGET_LINK_LIBRARIES(${appname} PRIVATE
            "${FRAMEWORK_${fwname}}/${fwname}")
        MESSAGE(STATUS "Framework ${fwname} found at ${FRAMEWORK_${fwname}}")
    endif()
endmacro(ADD_FRAMEWORK)
target_link_libraries(usbx PRIVATE objc)
add_framework(CoreFoundation usbx)
add_framework(IOKit usbx)
endif()
