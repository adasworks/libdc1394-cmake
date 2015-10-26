set(CSD ${CSD}/dc1394)

set(dc1394_sources "")
set(dc1394_link_libs "")

if(HAVE_LINUX)
	if(HAVE_LIBRAW1394)
		add_subdirectory(linux)
	endif()
	add_subdirectory(juju)
endif()
if(HAVE_MACOSX)
	install(FILES ${CSD}/macosx.h DESTINATION include/dc1394)
	list(APPEND dc1394_sources ${CSD}/macosx.c)
	add_subdirectory(macosx)
endif()
if(HAVE_WINDOWS AND NOT HAVE_NO_CMU1394)
	add_subdirectory(windows)
endif()
if(HAVE_LIBUSB)
	add_subdirectory(usb)
endif()
add_subdirectory(vendor)

list(APPEND dc1394_sources
	${CSD}/control.c
	${CSD}/control.h
	${CSD}/enumeration.c
	${CSD}/platform.h
	${CSD}/capture.c
	${CSD}/offsets.h
	${CSD}/format7.c
	${CSD}/register.c
	${CSD}/register.h
	${CSD}/utils.c
	${CSD}/utils.h
	${CSD}/internal.c
	${CSD}/internal.h
	${CSD}/conversions.c
	${CSD}/conversions.h
	${CSD}/bayer.c
	${CSD}/log.c
	${CSD}/log.h
	${CSD}/iso.c
	${CSD}/iso.h
	${linux_src}
	${juju_src}
	${macosx_src}
	${windows_src}
	${usb_src}
	${vendor_src}
)
add_library(libdc1394 ${dc1394_sources})

if(UNIX OR BUILD_SHARED_LIBS)
	set_target_properties(libdc1394 PROPERTIES OUTPUT_NAME "dc1394")
endif()

if(HAVE_LINUX AND HAVE_LIBRAW1394)
	list(APPEND dc1394_link_libs PRIVATE libraw1394) #linux/libdc1394-linux.la
endif()

target_include_directories(libdc1394 PUBLIC
	$<BUILD_INTERFACE:${CSD}>
	$<BUILD_INTERFACE:${CSD}/..>
	$<BUILD_INTERFACE:${PROJECT_BINARY_DIR}>
	$<INSTALL_INTERFACE:include>)
if(HAVE_WINDOWS)
	target_include_directories(libdc1394 PRIVATE
		${PROJECT_SOURCE_DIR}/1394camera646/1394camera)
endif()

target_link_libraries(libdc1394 ${dc1394_link_libs})
install(FILES
	${CSD}/dc1394.h
	${CSD}/types.h
	${CSD}/camera.h
	${CSD}/control.h
	${CSD}/capture.h
	${CSD}/video.h
	${CSD}/format7.h
	${CSD}/utils.h
	${CSD}/conversions.h
	${CSD}/register.h
	${CSD}/log.h
	${CSD}/iso.h
	DESTINATION include/dc1394
)

install(TARGETS libdc1394 EXPORT libdc1394-targets
	ARCHIVE DESTINATION lib
	LIBRARY DESTINATION lib
	RUNTIME DESTINATION bin)
install(EXPORT libdc1394-targets FILE libdc1394-config.cmake DESTINATION lib/cmake/libdc1394)
