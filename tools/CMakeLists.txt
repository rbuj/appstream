# CMakeLists for the AppStream CLI tool
project(appstream-cli)

find_package(GLIB2 REQUIRED)
pkg_check_modules(GIO2 REQUIRED gio-2.0)
find_library(M_LIB m)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}
		    ${CMAKE_BINARY_DIR}
		    ${CMAKE_SOURCE_DIR}/src
		    ${GLIB2_INCLUDE_DIR}
		    ${GIO2_INCLUDE_DIR}
)

set(ASCLI_SOURCES
	appstream-cli.c
	ascli-utils.h
	ascli-utils.c
	ascli-mdata-actions.h
	ascli-mdata-actions.c
	ascli-validate-actions.h
	ascli-validate-actions.c
)

# Add the appstream client tool
add_executable(appstreamcli ${ASCLI_SOURCES})
add_dependencies(appstreamcli appstream)

# the client tool uses API which is currently private...
# needs to be sorted out in future
add_definitions ("-DAS_COMPILATION")

target_link_libraries(appstreamcli
		${M_LIB}
		${GLIB2_LIBRARIES}
		${GIO2_LIBRARIES}
		appstream
)

install(TARGETS appstreamcli DESTINATION ${CMAKE_INSTALL_BINDIR})
