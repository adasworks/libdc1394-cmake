cmake_minimum_required(VERSION 2.8.12)
project(libdc1394 C)

include(CMakePrintHelpers)

set(CMAKE_DEBUG_POSTFIX d)

if(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/libdc1394)
    find_package(Git QUIET REQUIRED)
    execute_process(
        COMMAND ${GIT_EXECUTABLE}
            clone --depth 1
            	# git://git.code.sf.net/p/libdc1394/code # original project
            	git://git.code.sf.net/u/adasworks/libdc1394 # adasworks fork
            	${CMAKE_CURRENT_SOURCE_DIR}/libdc1394
        RESULT_VARIABLE r)
    if(r)
        message(FATAL_ERROR "git-cloning libdc1394 failed.")
    endif()
endif()

set(repo ${CMAKE_CURRENT_SOURCE_DIR}/libdc1394/libdc1394)

if(CMAKE_COMPILER_IS_GNUCC)
	if(CMAKE_VERSION VERSION_LESS 3.1)
		add_definitions(-std=c99)
	else()
		set(CMAKE_C_STANDARD 99)
		set(CMAKE_C_STANDARD_REQUIRED 1)
	endif()
endif()

include(CheckIncludeFile)
include(CheckCSourceCompiles)

set(HAVE_LIBRAW1394 0) #used in c am
set(HAVE_LIBSDL 0) #used in am

if(MINGW OR MSVC)
	set(ptrsize "")
	if(DEFINED CMAKE_SIZEOF_VOID_P)
		set(ptrsize ${CMAKE_SIZEOF_VOID_P})
	endif()
	if(DEFINED CMAKE_C_SIZEOF_DATA_PTR)
		if(ptrsize STREQUAL "")
			set(ptrsize ${CMAKE_C_SIZEOF_DATA_PTR})
		elseif(NOT CMAKE_C_SIZEOF_DATA_PTR EQUAL ptrsize)
			cmake_print_variables(CMAKE_SIZEOF_VOID_P CMAKE_C_SIZEOF_DATA_PTR)
			message(FATAL_ERROR "Ambigous values (see previous lines)")
		endif()
	endif()
	if(ptrsize STREQUAL "")
		message(FATAL_ERROR "Can't retrieve pointer size from CMAKE_SIZEOF_VOID_P and CMAKE_C_SIZEOF_DATA_PTR")
	elseif(NOT ptrsize EQUAL 4 AND NOT ptrsize EQUAL 8)
		message(FATAL_ERROR "Invalid pointer size: ${ptrsize}")
	endif()
	set(wlroot ${PROJECT_SOURCE_DIR}/libusb-1.0.20-windows)
	if(MINGW)
		set(wllib ${wlroot}/MinGW)
	elseif(MSVC)
		set(wllib ${wlroot}/MS)
	else()
		message(FATAL_ERROR "Internal error")
	endif()
	if(ptrsize EQUAL 4)
		set(wllib ${wllib}32/static)
	else()
		set(wllib ${wllib}64/static)
	endif()
	find_library(LIBUSB_LIBRARY NAMES usb-1.0 libusb-1.0 PATHS ${wllib} NO_DEFAULT_PATH)
	find_file(LIBUSB_HEADER NAMES libusb-1.0/libusb.h PATHS ${wlroot}/include NO_DEFAULT_PATH)
else()
	find_library(LIBUSB_LIBRARY NAMES usb-1.0 libusb-1.0)
	find_file(LIBUSB_HEADER NAMES libusb-1.0/libusb.h)
endif()

if(LIBUSB_LIBRARY AND LIBUSB_HEADER)
	message(STATUS "Found libusb: ${LIBUSB_LIBRARY}, ${LIBUSB_HEADER}")
	set(HAVE_LIBUSB 1)
	add_library(libdc1394::libusb IMPORTED UNKNOWN)
	get_filename_component(LIBUSB_INCLUDE_DIR "${LIBUSB_HEADER}" PATH)
	set_target_properties(libdc1394::libusb PROPERTIES
		IMPORTED_LOCATION ${LIBUSB_LIBRARY}
		INTERFACE_INCLUDE_DIRECTORIES ${LIBUSB_INCLUDE_DIR})
	if(NOT BUILD_SHARED_LIBS AND (MINGW OR MSVC))
		# found our own libusb, need to install it if static linking
		install(FILES ${LIBUSB_LIBRARY} DESTINATION lib/libdc1394)
	endif()
elseif(NOT HAVE_LIBUSB_LIBRARY AND NOT HAVE_LIBUSB_HEADER)
	message(STATUS "Building without libusb.")
	set(HAVE_LIBUSB 0)
else()
	cmake_print_variables(LIBUSB_LIBRARY LIBUSB_HEADER)
	message(FATAL_ERROR "Found an incorrectly installed libusb, see previous lines.")
endif()

if(CMAKE_SYSTEM_NAME MATCHES "Linux")
	set(HAVE_LINUX 1) # used in c am
endif()
if(APPLE AND NOT IOS)
	set(HAVE_MACOSX 1) # used in c am
endif()
check_include_file(netinet/in.h HAVE_NETINET_IN_H) # used in c
check_include_file(unistd.h HAVE_UNISTD_H) # used in c
check_include_file(pthread.h HAVE_PTHREAD_H) # used in c
check_include_file(Windows.h HAVE_WINDOWS_H) # used in c
check_include_file(sys/time.h HAVE_SYS_TIME_H) # used in c
find_package(OpenGL QUIET)
if(OPENGL_FOUND)
	set(HAVE_OPENGL 1) # used in am
endif()
check_include_file(sys/mman.h HAVE_SYS_MMAN_H) #used in c
set(HAVE_NO_CMU1394 1)
if(WIN32)
	set(HAVE_WINDOWS 1) # used in c am
endif()
if(HAVE_LIBUSB AND NOT HAVE_PTHREAD_H AND HAVE_WINDOWS_H)
	set(USE_WINDOWS_THREADS 1)
endif()
set(PACKAGE "libdc1394") # used in am
set(C99_RESTRICT "")
foreach(r restrict __restrict _Restrict __restrict__ __RESTRICT)
	check_c_source_compiles("int main() {int*${r} p;return 0;}" ${r}_IS_C99_RESTRICT)
	if(${r}_IS_C99_RESTRICT)
		set(C99_RESTRICT ${r})
		break()
	endif()
endforeach()
set(cfg ${PROJECT_BINARY_DIR}/config.h)
file(WRITE ${cfg} "")
# need to define size_t if you compiler does not support it: Define to `unsigned int' if <sys/types.h> does not define.
foreach(v
	HAVE_LIBRAW1394
	HAVE_LIBSDL
	HAVE_LIBUSB
	HAVE_LINUX
	HAVE_MACOSX
	HAVE_NETINET_IN_H
	HAVE_UNISTD_H
	HAVE_OPENGL
	HAVE_SYS_MMAN_H
	HAVE_WINDOWS
	HAVE_NO_CMU1394
	USE_WINDOWS_THREADS
	HAVE_SYS_TIME_H
	HAVE_WINDOWS_H
)
	if(${v})
		file(APPEND ${cfg} "#define ${v} 1\n")
	else()
		file(APPEND ${cfg} "/* #undef ${v} */\n")
	endif()
endforeach()
file(APPEND ${cfg} "#define PACKAGE \"${PACKAGE}\"\n")
if(C99_RESTRICT STREQUAL "restrict")
	file(APPEND ${cfg} "/* #undef restrict */\n")
else()
	file(APPEND ${cfg} "#define restrict ${C99_RESTRICT}\n")
endif()

check_include_file(linux/videodev.h HAVE_VIDEODEV) # used in am

# add lm

option(MAKE_EXAMPLES "build example programs" ON)

#    Makefile \
#    dc1394/Makefile \
#    dc1394/linux/Makefile \
#    dc1394/juju/Makefile \
#    dc1394/macosx/Makefile \
#    dc1394/windows/Makefile \
#    dc1394/usb/Makefile \
#    dc1394/vendor/Makefile \
#    examples/Makefile \


set(LINUXMSG "Disabled (Linux not detected)")
set(RAW1394MSG "Disabled (libraw1394 not found)")
set(MACOSXMSG "Disabled (Mac OS not detected)")
set(WINDOWSMSG "Disabled (Windows not detected)")
set(USBMSG "Disabled (libusb-1.0 not found)")

if(HAVE_LINUX)
	if(HAVE_LIBRAW1394)
		set(RAW1394MSG Enabled)
	endif()
	set(LINUXMSG Enabled)
endif()

if(HAVE_MACOSX)
  set(MACOSXMSG Enabled)
endif()

if(HAVE_WINDOWS)
	set(WINDOWSMSG Enabled)
endif()

if(HAVE_LIBUSB)
	set(USBMSG Enabled)
endif()

set(EXAMPLESMSG No)
set(SDLEXAMPLESMSG No)
set(XVEXAMPLESMSG No)
set(VIDEXAMPLESMSG No)

if(MAKE_EXAMPLES)
	set(EXAMPLESMSG Yes)
	if(HAVE_OPENGL)
    	if(HAVE_LIBSDL)
    		set(SDLEXAMPLESMSG Yes)
    	endif()
    endif()
	if(HAVE_XV)
    	set(XVEXAMPLESMSG Yes)
    endif()
    if(HAVE_VIDEODEV)
    	set(VIDEXAMPLESMSG Yes)
    endif()
endif()

message(STATUS "Linux support:                    ${LINUXMSG}")
if(LINUXMSG STREQUAL "Enabled")
	message(STATUS "Legacy drivers (libraw1394):      ${RAW1394MSG}")
endif()
message(STATUS "Mac OS X support:                 ${MACOSXMSG}")
message(STATUS "Windows support:                  ${WINDOWSMSG}")
message(STATUS "IIDC-over-USB support:            ${USBMSG}")
message(STATUS "Build examples:                   ${EXAMPLESMSG}")
if(EXAMPLESMSG)
	message(STATUS "Build SDL/OpenGL examples:        ${SDLEXAMPLESMSG}")
	message(STATUS "Build XV examples:                ${XVEXAMPLESMSG}")
	message(STATUS "Build V4L examples:               ${VIDEXAMPLESMSG}")
endif()

if(HAVE_LIBUSB AND NOT USE_WINDOWS_THREADS)
	if(CMAKE_VERSION VERSION_LESS 3.1)
		set(CMAKE_THREAD_PREFER_PTHREAD 1)
		find_package(Threads REQUIRED)
		if(NOT CMAKE_USE_PTHREADS_INIT)
			message(FATAL_ERROR "usb/control.c needs pthread, something else has been found.")
		endif()
		set(THREADS ${CMAKE_THREAD_LIBS_INIT})
	else()
		set(CMAKE_THREAD_PREFER_PTHREAD 1)
		set(THREADS_PREFER_PTHREAD_FLAG 1)
		find_package(Threads REQUIRED)
		if(NOT CMAKE_USE_PTHREADS_INIT)
			message(FATAL_ERROR "usb/control.c needs pthread, something else has been found: \"${CMAKE_THREAD_LIBS_INIT}\"")
		endif()
		set(THREADS Threads::Threads)
	endif()
endif()

add_definitions(-DHAVE_CONFIG)
set(CSD ${PROJECT_SOURCE_DIR}/libdc1394/libdc1394)
add_subdirectory(dc1394)
if(MAKE_EXAMPLES)
	add_subdirectory(examples)
endif()


