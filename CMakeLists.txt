# The MIT License (MIT)
#
# Copyright (c) 2020-2022 Scott Aron Bloom
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sub-license, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.21)
if(CMAKE_VERSION VERSION_LESS "3.7.0")
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
endif()

project(root-EdgeKiller) 

SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/SABUtils/Modules ${CMAKE_MODULE_PATH})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED true)
find_package(Threads)
find_package(Qt5 COMPONENTS Core Widgets REQUIRED)
find_package(Deploy REQUIRED)
find_package(CheckOpenSSL REQUIRED)
find_package(AddUnitTest REQUIRED)
find_package(Git REQUIRED)

GetGitInfo(${CMAKE_SOURCE_DIR} GIT_VERSION_INFO)

if(NOT EXISTS "${CMAKE_SOURCE_DIR}/VersionInfo.cmake")
	SET( VERSION_FILE_MAJOR_VERSION 0 )
	SET( VERSION_FILE_MINOR_VERSION 0 )
	SET( APP_NAME "Edge Killer" )
	SET( VENDOR   "Scott Aron Bloom" )
	SET( HOMEPAGE "www.towel42.com" )
	SET( EMAIL    "scott@towel42.com" )

	configure_file(
		"${CMAKE_SOURCE_DIR}/SABUtils/Modules/VersionInfo.cmake.in"
		"${CMAKE_SOURCE_DIR}/VersionInfo.cmake"
	)
endif()

include( VersionInfo.cmake )

string(TIMESTAMP BUILD_DATE "%m/%d/%Y")

CreateVersion(GIT_VERSION_INFO ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/SABUtils/Modules/Version.h.in
	${MAJOR_VERSION} ${MINOR_VERSION} ${APP_NAME} ${VENDOR} ${HOMEPAGE} ${EMAIL} ${BUILD_DATE})

option( gtest_force_shared_crt "Use shared ( DLL ) run-time lib even when Google Test is built as static lib." ON ) 

set_property( GLOBAL PROPERTY USE_FOLDERS ON )

file( REAL_PATH ~/bin/EdgeKiller CMAKE_INSTALL_PREFIX EXPAND_TILDE)
SET( SAB_ENABLE_TESTING ON )
include( ${CMAKE_SOURCE_DIR}/SABUtils/Modules/EnableGoogleTesting.cmake )
add_subdirectory( App/UnitTests )
add_subdirectory( App )

SET( CPACK_PACKAGE_VERSION_MAJOR ${MAJOR_VERSION} )
SET( CPACK_PACKAGE_VERSION_MINOR ${MINOR_VERSION} )
SET( CPACK_PACKAGE_VERSION_PATCH ${VERSION_FILE_PATCH_VERSION} )
SET( CPACK_PACKAGE_VERSION ${MAJOR_VERSION}.${MINOR_VERSION}.${VERSION_FILE_PATCH_VERSION})
SET( CPACK_PACKAGE_NAME ${APP_NAME} )
SET( CPACK_PACKAGE_VENDOR "${VENDOR} ${EMAIL}" )
if ( EXISTS ${CMAKE_SOURCE_DIR}/LICENSE )
	SET( CPACK_RESOURCE_FILE_LICENSE ${CMAKE_SOURCE_DIR}/LICENSE )
endif()

SET( CPACK_PACKAGE_HOMEPAGE_URL ${HOMEPAGE} )
set( CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${VENDOR}/${APP_NAME}" )
SET( CPACK_PACKAGE_DESCRIPTION "Application to redirect  edge calls to the system tool" )

SET( CPACK_GENERATOR ZIP NSIS )
if ( EXISTS ${CMAKE_SOURCE_DIR}/UI/resources/application.ico )
	SET( CPACK_NSIS_MUI_ICON ${CMAKE_SOURCE_DIR}/UI/resources/application.ico )
	SET( CPACK_NSIS_MUI_UNIICON ${CMAKE_SOURCE_DIR}/UI/resources/application.ico )
endif()
include( CPack )


