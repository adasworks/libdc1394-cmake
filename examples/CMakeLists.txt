set(CSD ${CSD}/examples)

set(A grab_gray_image grab_partial_image grab_color_image
	grab_color_image2 helloworld ladybug grab_partial_pvn
	basler_sff_info basler_sff_extended_data ladybug5capture)
set(B dc1394_reset_bus)

if(HAVE_LIBSDL)
	if(HAVE_OPENGL)
		list(APPEND A dc1394_iso)
	endif()
endif()

if(HAVE_LINUX)
	if(HAVE_VIDEODEV)
		list(APPEND B dc1394_vloopback)
	endif()
	if(HAVE_XV)
		list(APPEND A dc1394_multiview)
	endif()
endif()

foreach(p IN LISTS A B)
	if(p STREQUAL "dc1394_vloopback")
		add_executable(${p} ${CSD}/dc1394_vloopback.c ${CSD}/affine.c ${CSD}/affine.h)
	else()
		add_executable(${p} ${CSD}/${p}.c)
	endif()
	target_link_libraries(${p} libdc1394)
endforeach()

if(TARGET dc1394_multiview)
	target_link_libraries(dc1394_multiview PRIVATE x x_pre xv x11 x_extra)
endif()

if(TARGET dc1394_iso)
	target_include_directories(dc1394_iso PRIVATE ${OPENGL_INCLUDE_DIR})
	target_link_libraries(dc1394_iso PRIVATE sdl ${OPENGL_LIBRARIES})
endif()

