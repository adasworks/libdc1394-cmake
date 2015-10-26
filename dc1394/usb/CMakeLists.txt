set(CSD ${CSD}/usb)
set(usb_src
	${CSD}/control.c
	${CSD}/usb.h
	${CSD}/capture.c
	PARENT_SCOPE)

set(dc1394_link_libs ${dc1394_link_libs} PRIVATE libdc1394::libusb PARENT_SCOPE)

