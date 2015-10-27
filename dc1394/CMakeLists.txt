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
if(USE_WINDOWS_THREADS)
	list(APPEND dc1394_sources ${CSD}/pthread-windows.c ${CSD}/pthread-windows.h)
endif()
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
if(HAVE_WINDOWS AND NOT HAVE_NO_CMU1394)
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
install(EXPORT libdc1394-targets DESTINATION lib/cmake/libdc1394)

# Create config module
set(cfg ${PROJECT_BINARY_DIR}/hide/libdc1394-config.cmake)
file(WRITE ${cfg} "unset(libdc1394_FOUND)\n\n")

# libusb dependency
if(HAVE_LIBUSB AND NOT BUILD_SHARED_LIBS)
	file(APPEND ${cfg} "
		find_library(LIBDC1394_LIBUSB_LIBRARY NAMES usb-1.0 libusb-1.0 PATHS \${CMAKE_CURRENT_LIST_DIR}/../../libdc1394)
		if(NOT LIBDC1394_LIBUSB_LIBRARY)
			message(\"libusb not found (this libdc1394 is built with libusb support).\")
			set(libdc1394_FOUND 0)
		endif()
	")
	if(USE_WINDOWS_THREADS)
		# we use windows threads there
	elseif(CMAKE_VERSION VERSION_LESS 3.1)
		# threads is hopefully added with -lpthreads so we don't need to call
		# FindThreads here
	else()
		file(APPEND ${cfg} "
			set(CMAKE_THREAD_PREFER_PTHREAD 1)
			set(THREADS_PREFER_PTHREAD_FLAG 1)
			find_package(Threads)
			if(NOT CMAKE_THREAD_LIBS_INIT)
				message(\"libdc1394: no thread libs found\")
				set(libdc1394_FOUND 0)
			elseif(NOT TARGET Threads::Threads)
				message(\"Thread libs found but no Threads::Threads target created. You need CMake >= 3.1, current version: \${CMAKE_VERSION}\")
			elseif(NOT CMAKE_USE_PTHREADS_INIT)
				message(FATAL_ERROR \"libdc1394 needs pthread, something else has been found.\")
				set(libdc1394_FOUND 0)
			else()
				# here we can write what to do when threads found
			endif()
		")
	endif()

	file(APPEND ${cfg} "
		if(NOT DEFINED libdc1349_FOUND)
			add_library(libdc1394::libusb IMPORTED UNKNOWN)
			set_target_properties(libusb PROPERTIES IMPORTED_LOCATION \${LIBDC1394_LIBUSB_LIBRARY})
		endif()
	")
endif()

file(APPEND ${cfg} "
	if(NOT DEFINED libdc1394_FOUND)
		include(\${CMAKE_CURRENT_LIST_DIR}/libdc1394-targets.cmake)
	endif()
")
install(FILES ${PROJECT_BINARY_DIR}/hide/libdc1394-config.cmake DESTINATION lib/cmake/libdc1394)


