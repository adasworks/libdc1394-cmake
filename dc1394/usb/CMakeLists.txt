set(CSD ${CSD}/usb)
set(usb_src
	${CSD}/control.c
	${CSD}/usb.h
	${CSD}/capture.c
	PARENT_SCOPE)

if(NOT THREADS AND NOT (MSVC OR MINGW))
	message(FATAL_ERROR "The usb subdirectory needs THREADS")
endif()

set(dc1394_link_libs ${dc1394_link_libs} PRIVATE libdc1394::libusb ${THREADS} PARENT_SCOPE)

