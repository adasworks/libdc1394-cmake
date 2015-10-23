set(CSD ${CSD}/macosx)
set(macosx_src
	${CSD}/control.c
	${CSD}/capture.c
	${CSD}/macosx.h
	PARENT_SCOPE)

set(dc1394_link_libs ${dc1394_link_libs}
	PUBLIC "-framework CoreServices" "-framework IOKit"
	PARENT_SCOPE)


