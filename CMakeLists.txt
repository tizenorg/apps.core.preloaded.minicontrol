CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
PROJECT(minicontrol C)

SET(PREFIX ${CMAKE_INSTALL_PREFIX})
SET(EXEC_PREFIX "\${prefix}")
SET(PROJECT_NAME "${PROJECT_NAME}")
SET(LIBDIR "\${exec_prefix}/lib")
SET(INCLUDEDIR "\${prefix}/include/${PROJECT_NAME}")
SET(VERSION_MAJOR 0)
SET(VERSION "${VERSION_MAJOR}.0.1")

SET(CMAKE_SKIP_BUILD_RPATH true)

SET(INSTALL_HEADERS
	minicontrol-error.h
	minicontrol-type.h
	minicontrol-monitor.h
	minicontrol-provider.h
	minicontrol-viewer.h
)

SET(SUBMODULES
	minicontrol-provider
	minicontrol-viewer
	minicontrol-monitor
)
INCLUDE_DIRECTORIES(${CMAKE_SOURCE_DIR}/include)

INCLUDE(FindPkgConfig)
pkg_check_modules(pkgs REQUIRED
	dlog
	elementary
	evas
	ecore-evas
	dbus-1
	dbus-glib-1
)

FOREACH(flag ${pkgs_CFLAGS})
	SET(EXTRA_CFLAGS "${EXTRA_CFLAGS} ${flag}")
ENDFOREACH(flag)

SET(EXTRA_CFLAGS "${EXTRA_CFLAGS} -fvisibility=hidden -g -Wall")

SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${EXTRA_CFLAGS}")

ADD_DEFINITIONS("-DPREFIX=\"${PREFIX}\"")
ADD_DEFINITIONS("-DMINICTRL_USE_DLOG")

ADD_LIBRARY(${PROJECT_NAME}-inter STATIC
	src/minicontrol-internal.c
)
TARGET_LINK_LIBRARIES(${PROJECT_NAME}-inter ${pkgs_LDFLAGS})

FOREACH(lib_file ${SUBMODULES})
	ADD_LIBRARY(${lib_file} SHARED src/${lib_file}.c)
	TARGET_LINK_LIBRARIES(${lib_file} ${pkgs_LDFLAGS} ${PROJECT_NAME}-inter)
	SET_TARGET_PROPERTIES(${lib_file} PROPERTIES SOVERSION ${VERSION_MAJOR})
	SET_TARGET_PROPERTIES(${lib_file} PROPERTIES VERSION ${VERSION})
	INSTALL(TARGETS ${lib_file} DESTINATION lib COMPONENT RuntimeLibraries)
ENDFOREACH(lib_file)

FOREACH(pcfile ${SUBMODULES})
	CONFIGURE_FILE(${pcfile}.pc.in ${pcfile}.pc @ONLY)
	SET_DIRECTORY_PROPERTIES(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${pcfile}.pc")
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/${pcfile}.pc DESTINATION lib/pkgconfig)
ENDFOREACH(pcfile)

FOREACH(hfile ${INSTALL_HEADERS})
	INSTALL(FILES ${CMAKE_SOURCE_DIR}/include/${hfile} DESTINATION include/${PROJECT_NAME})
ENDFOREACH(hfile)

