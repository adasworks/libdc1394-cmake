set(CSD ${CSD}/linux)

set(linux_src
	${CSD}/control.c
	${CSD}/capture.c
	${CSD}/capture.h
	${CSD}/kernel-video1394.h
	${CSD}/linux.h
	PARENT_SCOPE)

if(HAVE_LIBRAW1394)
	install(
		FILES
			${CSD}/capture.h
			${CSD}/control.h
		DESTINATION
			include/dc1394/linux)
endif()

set(dc1394_link_libs ${dc1394_link_libs}
	libraw1394
	PARENT_SCOPE)

