# This is a cmake script. Process it with the CMake gui or command line utility
# to produce makefiles / Visual Studio project files on Mac OS X and Windows.
#
# To configure the build options either use the CMake gui, or run the command
# line utility including the "-i" option.

set(CMAKE_LEGACY_CYGWIN_WIN32 0)

project(mosquitto)

cmake_minimum_required(VERSION 2.8)
# Only for version 3 and up. cmake_policy(SET CMP0042 NEW)

set(PROJECT_VERSION_MAJOR 1)
set(PROJECT_VERSION_MINOR 6)
set(PROJECT_VERSION_PATCH 10)
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})

add_definitions (-DCMAKE -DVERSION=\"${PROJECT_VERSION}\")

if (WIN32)
	add_definitions("-D_CRT_SECURE_NO_WARNINGS")
	add_definitions("-D_CRT_NONSTDC_NO_DEPRECATE")
endif (WIN32)

include(GNUInstallDirs)

option(WITH_BUNDLED_DEPS "Build with bundled dependencies?" ON)
option(WITH_TLS
	"Include SSL/TLS support?" ON)
option(WITH_TLS_PSK
	"Include TLS-PSK support (requires WITH_TLS)?" ON)
option(WITH_EC
	"Include Elliptic Curve support (requires WITH_TLS)?" ON)
if (WITH_TLS)
	find_package(OpenSSL REQUIRED)
	add_definitions("-DWITH_TLS")

	if (WITH_TLS_PSK)
		add_definitions("-DWITH_TLS_PSK")
	endif (WITH_TLS_PSK)

	if (WITH_EC)
		add_definitions("-DWITH_EC")
	endif (WITH_EC)
else (WITH_TLS)
	set (OPENSSL_INCLUDE_DIR "")
endif (WITH_TLS)

option(WITH_SOCKS "Include SOCKS5 support?" ON)
if (WITH_SOCKS)
	add_definitions("-DWITH_SOCKS")
endif (WITH_SOCKS)

option(WITH_SRV "Include SRV lookup support?" OFF)

option(WITH_STATIC_LIBRARIES "Build static versions of the libmosquitto/pp libraries?" OFF)
option(WITH_PIC "Build the static library with PIC (Position Independent Code) enabled archives?" OFF)

option(WITH_THREADING "Include client library threading support?" ON)
if (WITH_THREADING)
	add_definitions("-DWITH_THREADING")
	if (WIN32)
		if (CMAKE_CL_64)
			set (PTHREAD_LIBRARIES C:\\pthreads\\Pre-built.2\\lib\\x64\\pthreadVC2.lib)
		else (CMAKE_CL_64)
			set (PTHREAD_LIBRARIES C:\\pthreads\\Pre-built.2\\lib\\x86\\pthreadVC2.lib)
		endif (CMAKE_CL_64)
		set (PTHREAD_INCLUDE_DIR C:\\pthreads\\Pre-built.2\\include)
	else (WIN32)
		find_library(LIBPTHREAD pthread)
		if (LIBPTHREAD)
			set (PTHREAD_LIBRARIES pthread)
		else (LIBPTHREAD)
			set (PTHREAD_LIBRARIES "")
		endif()
		set (PTHREAD_INCLUDE_DIR "")
	endif (WIN32)
else (WITH_THREADING)
	set (PTHREAD_LIBRARIES "")
	set (PTHREAD_INCLUDE_DIR "")
endif (WITH_THREADING)

option(DOCUMENTATION "Build documentation?" ON)

option(WITH_DLT "Include DLT support?" OFF)
message(STATUS "WITH_DLT = ${WITH_DLT}")
if (WITH_DLT)
	#find_package(DLT REQUIRED)
	find_package(PkgConfig)
	pkg_check_modules(DLT "automotive-dlt >= 2.11")
	add_definitions("-DWITH_DLT")
endif (WITH_DLT)

# ========================================
# Include projects
# ========================================

add_subdirectory(lib)
add_subdirectory(client)
add_subdirectory(src)
if (DOCUMENTATION)
	add_subdirectory(man)
endif (DOCUMENTATION)

# ========================================
# Install config file
# ========================================
install(FILES mosquitto.conf aclfile.example pskfile.example pwfile.example
    DESTINATION "${CMAKE_INSTALL_SYSCONFDIR}/mosquitto"
    COMPONENT Library )



# ========================================
# Testing
# ========================================
enable_testing()

# =========================================
# Package creation using cpack (Balluff)
# =========================================

set( CPACK_PACKAGE_NAME ${PROJECT_NAME} )
set( CPACK_PACKAGE_VENDOR "Eclipse Foundation" )
set( CPACK_PACKAGE_DESCRIPTION_SUMMARY
    "Mosquitto is an open source implementation of a server for version 5.0, 3.1.1, and 3.1 of the MQTT protocol."
    )
set( CPACK_PACKAGE_HOMEPAGE_URL "https://github.com/eclipse/mosquitto" )
set( CPACK_PACKAGE_CONTACT      "https://github.com/eclipse/mosquitto" )
set( CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR} )
set( CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR} )
set( CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH} )
set( CPACK_PACKAGE_VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH} )

set( CPACK_PACKAGE_INSTALL_DIRECTORY ${PROJECT_NAME} )
set( CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt" )
set( CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/readme.md" )

set( CPACK_COMPONENT_Library_DISPLAY_NAME "Mosquitto Library" )
set( CPACK_COMPONENT_Library_DESCRIPTION "The Mosquitto binary library." )
set( CPACK_COMPONENT_Library_REQUIRED 1 )
set( CPACK_COMPONENT_Devel_DISPLAY_NAME "Mosquitto Development Files" )
set( CPACK_COMPONENT_Devel_DESCRIPTION "Development files for compiling against Mosquitto." )
set( CPACK_COMPONENT_Devel_REQUIRED 0 )

if( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" )

    if ( "${CPACK_PACKAGE_ARCHITECTURE}" STREQUAL "" )
        # Note: the architecture should default to the local architecture, but it
        # in fact comes up empty.  We call `uname -m` to ask the kernel instead.
        EXECUTE_PROCESS( COMMAND uname -m COMMAND tr -d '\n' OUTPUT_VARIABLE CPACK_PACKAGE_ARCHITECTURE )
    endif()

    set( CPACK_INCLUDE_TOPLEVEL_DIRECTORY 1 )
    set( CPACK_PACKAGE_RELEASE 1 )


    # RPM - https://cmake.org/cmake/help/latest/cpack_gen/rpm.html
    set( CPACK_RPM_PACKAGE_RELEASE ${CPACK_PACKAGE_RELEASE} )
    set( CPACK_RPM_PACKAGE_ARCHITECTURE ${CPACK_PACKAGE_ARCHITECTURE} )
    set( CPACK_RPM_PACKAGE_DESCRIPTION ${CPACK_PACKAGE_DESCRIPTION_SUMMARY} )
    set( CPACK_RPM_PACKAGE_URL ${CPACK_PACKAGE_HOMEPAGE_URL} )
    set( CPACK_RPM_PACKAGE_LICENSE "EPL-1 AND EDL-1" )
    set( CPACK_RPM_COMPONENT_INSTALL 1 )
    set( CPACK_RPM_COMPRESSION_TYPE "xz" )
    set( CPACK_RPM_PACKAGE_AUTOPROV 1 )

    set( CPACK_RPM_Library_PACKAGE_SUMMARY ${CPACK_COMPONENT_Library_DESCRIPTION} )
    set( CPACK_RPM_Library_PACKAGE_ARCHITECTURE ${CPACK_RPM_PACKAGE_ARCHITECTURE} )
    set( CPACK_RPM_Library_PACKAGE_NAME ${CPACK_PACKAGE_NAME} )
    set( CPACK_RPM_Library_FILE_NAME "RPM-DEFAULT" )

    set( CPACK_RPM_Devel_PACKAGE_REQUIRES "cmake >= ${CMAKE_MINIMUM_REQUIRED_VERSION},mosquitto >= ${CPACK_PACKAGE_VERSION}" )
    set( CPACK_RPM_Devel_PACKAGE_SUMMARY ${CPACK_COMPONENT_Devel_DESCRIPTION} )
    set( CPACK_RPM_Devel_PACKAGE_ARCHITECTURE ${CPACK_RPM_PACKAGE_ARCHITECTURE} )
    set( CPACK_RPM_Devel_PACKAGE_NAME "${CPACK_PACKAGE_NAME}-devel" )
    set( CPACK_RPM_Devel_FILE_NAME "RPM-DEFAULT" )

    set( CPACK_RPM_Clients_PACKAGE_REQUIRES "cmake >= ${CMAKE_MINIMUM_REQUIRED_VERSION},mosquitto >= ${CPACK_PACKAGE_VERSION}" )
    set( CPACK_RPM_Clients_PACKAGE_SUMMARY ${CPACK_COMPONENT_Clients_DESCRIPTION} )
    set( CPACK_RPM_Clients_PACKAGE_ARCHITECTURE ${CPACK_RPM_PACKAGE_ARCHITECTURE} )
    set( CPACK_RPM_Clients_PACKAGE_NAME "${CPACK_PACKAGE_NAME}-clients" )
    set( CPACK_RPM_Clients_FILE_NAME "RPM-DEFAULT" )

    # DEB - https://cmake.org/cmake/help/latest/cpack_gen/deb.html
    set( CPACK_DEBIAN_PACKAGE_RELEASE ${CPACK_PACKAGE_RELEASE} )
    set( CPACK_DEBIAN_PACKAGE_HOMEPAGE ${CPACK_PACKAGE_HOMEPAGE_URL} )
    set( CPACK_DEB_COMPONENT_INSTALL 1 )
    set( CPACK_DEBIAN_COMPRESSION_TYPE "xz")

    if ( ${CPACK_PACKAGE_ARCHITECTURE} STREQUAL "x86_64" )
        set( CPACK_DEBIAN_PACKAGE_ARCHITECTURE "amd64" )  # DEB doesn't always use the kernel's arch name
    else()
        set( CPACK_DEBIAN_PACKAGE_ARCHITECTURE ${CPACK_PACKAGE_ARCHITECTURE} )
    endif()

    set( CPACK_DEBIAN_FILE_NAME "DEB-DEFAULT" ) # Use default naming scheme

    set( CPACK_DEBIAN_LIBRARY_PACKAGE_NAME "${CPACK_PACKAGE_NAME}" )
    set( CPACK_DEBIAN_LIBRARY_PACKAGE_SHLIBDEPS 1 )

    set( CPACK_DEBIAN_DEVEL_PACKAGE_DEPENDS "cmake (>= ${CMAKE_MINIMUM_REQUIRED_VERSION}), mosquitto (>= ${CPACK_PACKAGE_VERSION})" )
    set( CPACK_DEBIAN_DEVEL_PACKAGE_NAME "${CPACK_PACKAGE_NAME}-dev" )

    set( CPACK_DEBIAN_CLIENTS_PACKAGE_DEPENDS "cmake (>= ${CMAKE_MINIMUM_REQUIRED_VERSION}), mosquitto (>= ${CPACK_PACKAGE_VERSION})" )
    set( CPACK_DEBIAN_CLIENTS_PACKAGE_NAME "${CPACK_PACKAGE_NAME}-clients" )


elseif( ${CMAKE_HOST_WIN32} )
    set( CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON )
    set( CPACK_NSIS_DISPLAY_NAME ${PROJECT_NAME} )
    set( CPACK_NSIS_PACKAGE_NAME ${PROJECT_NAME} )
    set( CPACK_NSIS_URL_INFO_ABOUT ${CPACK_PACKAGE_HOMEPAGE_URL} )
endif()

# This must always be last!
include(CPack)

