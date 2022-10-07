cmake_minimum_required(VERSION 3.22)
project(idevicerestore)
set(ignoreMe "${NO_PKGCFG}")
set(ignoreMe "${ASAN}")
set(ignoreMe "${ARCH}")
if(NOT DEFINED SUBPROJECT_BUILD)
	set(ASAN_FLAG "")
	if(DEFINED ASAN OR "$ENV{ASAN}" MATCHES "1")
		set(ASAN 1)
		set(ASAN_FLAG -fsanitize=address)
		set(CMAKE_C_LINK_LIBRARY_FLAG "${ASAN_FLAG}")
	endif()
	if("${CMAKE_HOST_SYSTEM_NAME}" MATCHES "Darwin")
		if(NOT DEFINED NO_XCODE AND NOT DEFINED ENV{NO_XCODE})
			execute_process(COMMAND xcrun --find clang WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}" OUTPUT_VARIABLE CC OUTPUT_STRIP_TRAILING_WHITESPACE)
			execute_process(COMMAND xcrun --find ar WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}" OUTPUT_VARIABLE AR OUTPUT_STRIP_TRAILING_WHITESPACE)
			execute_process(COMMAND xcrun --find ld WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}" OUTPUT_VARIABLE LD OUTPUT_STRIP_TRAILING_WHITESPACE)
			execute_process(COMMAND xcrun --show-sdk-path WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}" OUTPUT_VARIABLE SYSROOT OUTPUT_STRIP_TRAILING_WHITESPACE)
			set(CMAKE_C_COMPILER "${CC}")
		endif()
		if(NOT DEFINED ARCH)
			set(ARCH "$ENV{ARCH}")
			if(ARCH STREQUAL "")
				set(ARCH "${CMAKE_SYSTEM_PROCESSOR}")
			endif()
		endif()
		if(NOT DEFINED MINVER OR NOT DEFINED "$ENV{MINVER}" AND DEFINED NO_PKGCFG AND "$ENV{NO_PKGCFG}" MATCHES "1")
			if("${ARCH}" STREQUAL "x86_64" OR "$ENV{ARCH}" STREQUAL "x86_64")
				set(MINVER -mmacosx-version-min=10.12)
				set(CMAKE_OSX_DEPLOYMENT_TARGET 10.12)
			else()
				set(MINVER -mmacosx-version-min=11.0)
				set(CMAKE_OSX_DEPLOYMENT_TARGET 11.0)
			endif()
		endif()
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -arch ${ARCH} -isysroot ${SYSROOT} ${MINVER} -fembed-bitcode=off -flto=thin -pthread -mrelax-all -std=gnu17 ${ASAN_FLAG}")
	elseif("${CMAKE_HOST_SYSTEM_NAME}" MATCHES "Linux")
		set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wl,--allow-multiple-definition -pthread -mrelax-all -std=gnu17 ${ASAN_FLAG}")
	endif()
endif()
set(CMAKE_C_FLAGS_RELEASE "-Os -DNDEBUG")
set(CMAKE_C_FLAGS_DEBUG "-g -O0 -DDEBUG")
set(CMAKE_C_STANDARD 17)
add_library(idevicerestore
        src/idevicerestore.c
        src/common.c
        src/tss.c
        src/fls.c
        src/mbn.c
        src/img3.c
        src/img4.c
        src/ftab.c
        src/ipsw.c
        src/normal.c
        src/dfu.c
        src/recovery.c
        src/restore.c
        src/asr.c
        src/fdr.c
        src/limera1n.c
        src/download.c
        src/locking.c)
target_include_directories(idevicerestore PRIVATE
        "${CMAKE_SOURCE_DIR}/dep_root/include"
        src)
target_link_directories(idevicerestore PRIVATE
        "${CMAKE_SOURCE_DIR}/dep_root/lib")
if(NOT DEFINED NO_PKGCFG AND NOT "$ENV{NO_PKGCFG}" MATCHES "1")
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(DEPS REQUIRED
            libzip
            libssl
            libplist-2.0
            libimobiledevice-glue-1.0
            libimobiledevice-1.0
            libirecovery-1.0)
    target_include_directories(idevicerestore PRIVATE "${DEPS_INCLUDE_DIRS}")
endif()
execute_process(COMMAND git remote get-url origin WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/idevicerestore/src" OUTPUT_VARIABLE PACKAGE_URL1 OUTPUT_STRIP_TRAILING_WHITESPACE)
string(REPLACE ".git" "" PACKAGE_URL PACKAGE_URL1)
add_definitions(
        -DIDEVICERESTORE_NOMAIN=1
        -DHAVE_OPENSSL=1
        -DPACKAGE_VERSION="1.0.0"
        -DPACKAGE_URL="${PACKAGE_URL}"
        -DPACKAGE_BUGREPORT="${PACKAGE_URL}/issues"
        -DPACKAGE_NAME="idevicerestore")
if(NOT "${CMAKE_SYSTEM_NAME}" MATCHES "MSYS" AND NOT "${CMAKE_SYSTEM_NAME}" MATCHES "Windows")
    add_definitions(-DHAVE_REALPATH=1)
endif()
if(NOT DEFINED SUBPROJECT_BUILD)
	install(TARGETS idevicerestore
		DESTINATION "${CMAKE_INSTALL_PREFIX}/lib")
endif()

