cmake_minimum_required(VERSION 3.19...3.24 FATAL_ERROR)
set(ignoreMe "${NO_PKGCFG} ${ASAN} ${ASAN_FLAGS} ${ARCH}")
option(SUBPROJECT_BUILD "Enables submodules to build as statically internally linked libs instead of binaries/tools" OFF)
option(ASAN "Enable AddressSanatizer" OFF)
option(NO_PKGCFG "Disable pkgconfig searching of libs, use dep_root for linking" OFF)
if(NOT SUBPROJECT_BUILD)
	if(ARCH STREQUAL "")
		set(ARCH "${CMAKE_SYSTEM_PROCESSOR}")
	endif()

	if("${CMAKE_HOST_SYSTEM_NAME}" MATCHES "Darwin")
		set(CMAKE_OSX_ARCHITECTURES="${ARCH}")
		if((NOT DEFINED MINVER) AND NO_PKGCFG)
			if("${ARCH}" STREQUAL "x86_64")
				set(CMAKE_OSX_DEPLOYMENT_TARGET 10.12)
			else()
				set(CMAKE_OSX_DEPLOYMENT_TARGET 11.0)
			endif()
		endif()
		if(DEFINED MINVER)
			set(CMAKE_OSX_DEPLOYMENT_TARGET "${MINVER}")
		endif()
		set(PROJECT_C_FLAGS "-arch ${ARCH} -fembed-bitcode=off -pthread -mrelax-all -Wno-deprecated-declarations -Wno-int-conversion")
		if(NOT "${CMAKE_BUILD_TYPE}" MATCHES "Debug" AND NOT "${CMAKE_BUILD_TYPE}" MATCHES "RelWithDebInfo")
			set(PROJECT_C_FLAGS "-flto=thin ${PROJECT_C_FLAGS}")
		endif()
	elseif("${CMAKE_HOST_SYSTEM_NAME}" MATCHES "Linux")
		set(PROJECT_C_FLAGS "-Wl,--allow-multiple-definition -pthread -mrelax-all -Wno-deprecated-declarations -Wno-int-conversion")
	elseif("${CMAKE_HOST_SYSTEM_NAME}" MATCHES "MSYS" OR "${CMAKE_HOST_SYSTEM_NAME}" MATCHES "Windows")
		set(PROJECT_C_FLAGS "-Wl,--allow-multiple-definition -pthread -fPIC -Wno-deprecated-declarations -Wno-int-conversion -DCURL_STATICLIB -Bstatic -O0")
	endif()

	project(idevicerestore VERSION 1.0.0 LANGUAGES C)

	if(ASAN)
		set(ASAN ON)
		set(ASAN_FLAGS "-fsanitize=address -fsanitize-address-use-after-scope -fno-omit-frame-pointer ${ASAN_FLAGS}")
		set(CMAKE_C_LINK_LIBRARY_FLAG "${CMAKE_C_LINK_LIBRARY_FLAG} ${ASAN_FLAGS}")
		set(CMAKE_MACOSX_RPATH 1)
		set(CMAKE_INSTALL_RPATH_USE_LINK_PATH 1)
		set(CMAKE_BUILD_RPATH
				"/usr/local/lib"
				"/usr/lib"
				"/opt/procursus/lib"
				"/Library/Developer/CommandLineTools/usr/lib/clang/14.0.0/lib/darwin"
				"/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/clang/14.0.0/lib/darwin/"
				"/Applications/Xcode_14.1.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/clang/14.0.0/lib/darwin/"
				"/Applications/Xcode_14.2.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/clang/14.0.0/lib/darwin/")
		#hardcoded because actions never updates xcode
	else()
		set(ASAN_FLAGS "")
	endif()

	set(CMAKE_C_FLAGS_MINSIZEREL_INIT "${ADDITIONAL_C_FLAGS} ${PROJECT_C_FLAGS} ${CMAKE_C_FLAGS_MINSIZEREL_INIT}")
	set(CMAKE_C_FLAGS_RELEASE_INIT "${ADDITIONAL_C_FLAGS} ${PROJECT_C_FLAGS} ${CMAKE_C_FLAGS_RELEASE_INIT}")
	set(CMAKE_C_FLAGS_RELWITHDEBINFO_INIT "${ADDITIONAL_C_FLAGS} ${PROJECT_C_FLAGS} ${CMAKE_C_FLAGS_RELWITHDEBINFO_INIT}")
	set(CMAKE_C_FLAGS_DEBUG_INIT "${ADDITIONAL_C_FLAGS} ${PROJECT_C_FLAGS} ${CMAKE_C_FLAGS_DEBUG_INIT} ${ASAN_FLAGS}")

	set(CMAKE_C_FLAGS_MINSIZEREL "${CMAKE_C_FLAGS_MINSIZEREL_INIT}")
	set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE_INIT}")
	set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO_INIT}")
	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG_INIT}")

	set(CMAKE_C_STANDARD 17)
else()
	project(idevicerestore VERSION 1.0.0 LANGUAGES C)
endif()


if(NO_PKGCFG)
	find_package(PkgConfig REQUIRED FATAL_ERROR)
endif()

add_library(idevicerestore
		src/idevicerestore.c
		src/common.c
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
		src/locking.c
		src/ace3.c)
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
		-DHAVE_IDEVICE_E_TIMEOUT=1
		-DHAVE_RESTORE_E_RECEIVE_TIMEOUT=1
		-DHAVE_ENUM_IDEVICE_CONNECTION_TYPE=1
		-DHAVE_REVERSE_PROXY=1
		-DHAVE_OPENSSL=1
		-DPACKAGE_VERSION="${CMAKE_PROJECT_VERSION}"
		-DPACKAGE_URL="${PACKAGE_URL}"
		-DPACKAGE_BUGREPORT="${PACKAGE_URL}/issues"
		-DPACKAGE_NAME="idevicerestore")
if(NOT "${CMAKE_SYSTEM_NAME}" MATCHES "MSYS" AND NOT "${CMAKE_SYSTEM_NAME}" MATCHES "Windows")
	add_definitions(
			-DHAVE_REALPATH=1
			-DHAVE_REVERSE_PROXY=1)
endif()
if(NOT DEFINED SUBPROJECT_BUILD)
	install(TARGETS idevicerestore
			DESTINATION "${CMAKE_INSTALL_PREFIX}/lib")
endif()

#SET(CMAKE_EXPORT_COMPILE_COMMANDS 1)
#get_cmake_property(_variableNames VARIABLES)
#list (SORT _variableNames)
#foreach (_variableName ${_variableNames})
#	message(STATUS "${_variableName}=${${_variableName}}")
#endforeach()

