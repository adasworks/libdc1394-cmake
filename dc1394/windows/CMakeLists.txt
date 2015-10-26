set(CSD ${CSD}/windows)
set(windows_src
	${CSD}/control.c ${CSD}/capture.c ${CSD}/platform_windows.h
	PARENT_SCOPE)

set(dc1394_link_libs ${dc1394_link_libs} PRIVATE Setupapi PARENT_SCOPE)

