set(CSD ${PROJECT_SOURCE_DIR}/libdc1394/libdc1394/dc1394)

#add_subdirectory(linux)
#add_subdirectory(juju)
#add_subdirectory(macosx)
#add_subdirectory(windows)
#add_subdirectory(usb)
#add_subdirectory(vendor)

set(srcs
	control.c
	control.h
	enumeration.c
	platform.h
	capture.c
	offsets.h
	format7.c
	register.c
	register.h
	utils.c
	utils.h
	internal.c
	internal.h
	conversions.c
	conversions.h
	bayer.c
	log.c
	log.h
	iso.c
	iso.h
)
foreach(f IN LISTS srcs)
	list(APPEND libdc1394_la_SOURCES ${CSD}/${f})
endforeach()

add_library(libdc1394 ${libdc1394_la_SOURCES})

if(UNIX OR BUILD_SHARED_LIBS)
	set_target_properties(libdc1394 PROPERTIES OUTPUT_NAME "dc1394")
endif()

if(HAVE_LINUX)
	if(HAVE_LIBRAW1394)
  		target_link_libraries(libdc1394 PRIVATE libraw1394) #linux/libdc1394-linux.la
  	endif()
	target_link_libraries(libdc1394 PRIVATE libdc1394-juju) #juju/libdc1394-juju.la
endif()

if(HAVE_MACOSX)
	target_sources(libdc1394 PRIVATE ${CSD}/macosx.c)
	target_link_libraries(libdc1394 PRIVATE libdc1394-macosx)
endif()

if(HAVE_WINDOWS)
	target_link_libraries(libdc1394 PRIVATE libdc1394-windows)
endif()

if(HAVE_LIBUSB)
	target_link_libraries(libdc1394 PRIVATE libdc1394-usb)
endif()

target_link_libraries(libdc1394 PRIVATE libdc1394-vendor)
target_include_directories(libdc1394 PUBLIC
	$<BUILD_INTERFACE:${CSD}>
	$<INSTALL_INTERFACE:include>)
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
	DESTINATION include/dc1394)
install(TARGETS libdc1394 EXPORT libdc1394-targets
	ARCHIVE DESTINATION lib
	LIBRARY DESTINATION lib
	RUNTIME DESTINATION bin)
install(EXPORT libdc1394-targets FILE libdc1394-config.cmake DESTINATION lib/cmake/libdc1394)
if(HAVE_MACOSX)
	install(FILES ${CSD}/macosx.h DESTINATION include/dc1394)
endif()
