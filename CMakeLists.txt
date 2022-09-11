cmake_minimum_required(VERSION 3.15)
project(ptouch-print C)

# Configure CMake
set(CMAKE_C_STANDARD 11)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

# Configure required dependencies
find_package(Gettext REQUIRED)
find_package(GD REQUIRED)
find_package(Git REQUIRED)
find_package(PkgConfig REQUIRED)
find_package(Intl REQUIRED)

pkg_check_modules(LIBUSB REQUIRED libusb-1.0)

# Configure project executable
add_executable(ptouch-print)

target_sources(ptouch-print PUBLIC
	version.h
	include/ptouch.h
	include/gettext.h
	src/libptouch.c
	src/ptouch-print.c
)

# Configure compiler
target_compile_options(ptouch-print PUBLIC
	-g
	-Wall
	-Wextra
	-Wunused
	-O3
	-fPIC
)

target_compile_definitions(ptouch-print PUBLIC
	LOCALEDIR="${CMAKE_INSTALL_LOCALEDIR}"
	USING_CMAKE=1
	PACKAGE="ptouch-print"
)

target_include_directories(ptouch-print PUBLIC
	include
	${GD_INCLUDE_DIR}
	${LIBUSB_INCLUDE_DIRS}
	${CMAKE_BINARY_DIR}
	${Intl_INCLUDE_DIRS}
)

# Configure linker
target_link_libraries(ptouch-print
	${GD_LIBRARIES}
	${LIBUSB_LIBRARIES}
	${LIBUSB_LINK_LIBRARIES}
	${Intl_LIBRARY}
)

# Add a custom command that produces version.h, plus a dummy output that's
# not produced, in order to force gitversion.cmake to always be re-run
# before the build
ADD_CUSTOM_COMMAND(
	OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/version.h
		${CMAKE_BINARY_DIR}/_version.h
	COMMAND ${CMAKE_COMMAND} -P
		${CMAKE_CURRENT_SOURCE_DIR}/cmake/gitversion.cmake)

include(GNUInstallDirs)
set(CMAKE_INSTALL_PREFIX "/usr")

install(TARGETS ptouch-print
)

INSTALL(FILES ${CMAKE_CURRENT_SOURCE_DIR}/ptouch-print.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1)
