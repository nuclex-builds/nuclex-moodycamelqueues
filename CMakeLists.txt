#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
	MoodyCamelQueues
	VERSION 1.0.4
	DESCRIPTION "Fast, lock-free queues for C++11"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build-concurrentqueue)
	message(STATUS "Build directory not present, downloading sources")

	file(
		DOWNLOAD https://github.com/cameron314/concurrentqueue/archive/refs/tags/v1.0.4.tar.gz
		${PROJECT_SOURCE_DIR}/downloads/v1.0.4.tar.gz
		SHOW_PROGRESS
		EXPECTED_HASH SHA256=87fbc9884d60d0d4bf3462c18f4c0ee0a9311d0519341cac7cbd361c885e5281
	)
	list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
	if(NOT ${STATUS_CODE} EQUAL 0)
		list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
		file(REMOVE ${PROJECT_SOURCE_DIR}/downloads/v1.0.4.tar.gz)
		message(FATAL_ERROR "Could not download MoodyCamelQueues: ${ERROR_MESSAGE}")
	endif()

	message(STATUS "Extracting MoodyCamelQueues sources")
	file(
		ARCHIVE_EXTRACT
		INPUT ${PROJECT_SOURCE_DIR}/downloads/v1.0.4.tar.gz
		DESTINATION ${CMAKE_BINARY_DIR}/extract
	)
	file(
		RENAME
		${CMAKE_BINARY_DIR}/extract/MoodyCamelQueues-2.7.2
		${PROJECT_SOURCE_DIR}/build-concurrentqueue
	)
endif()

# -------------------------------------------------------------------------------------------------

#configure_file(
#	"${PROJECT_SOURCE_DIR}/example.h"
#	"${PROJECT_SOURCE_DIR}/build-concurrentqueue/include/moodycamelqueues/"
#	COPYONLY
#)

# -------------------------------------------------------------------------------------------------

set(
	sourceFiles
	""
)
set(
	headerFiles
	"build-concurrentqueue/blockingconcurrentqueue.h"
	"build-concurrentqueue/concurrentqueue.h"
	"build-concurrentqueue/internal/concurrentqueue_internal_debug.h"
	"build-concurrentqueue/lightweightsemaphore.h"
)

# -------------------------------------------------------------------------------------------------

add_library(MoodyCamelQueues STATIC)
add_library(MoodyCamelQueues::Static ALIAS MoodyCamelQueues)

target_compile_definitions(
	MoodyCamelQueues
	PUBLIC MOODYCAMELQUEUES_STATIC
	PRIVATE MOODYCAMELQUEUES_EXPORTS
)

target_include_directories(
	MoodyCamelQueues
	PUBLIC "build/include"
)

target_sources(
	MoodyCamelQueues
	PUBLIC ${headerFiles}
	PRIVATE ${sourceFiles}
)

set_target_properties(MoodyCamelQueues PROPERTIES PREFIX "")
set_target_properties(MoodyCamelQueues PROPERTIES OUTPUT_NAME "moodycamelqueues")

# -------------------------------------------------------------------------------------------------

install(
	TARGETS MoodyCamelQueues
	ARCHIVE	DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)

# -------------------------------------------------------------------------------------------------

file(
	WRITE "${PROJECT_SOURCE_DIR}/MoodyCamelQueuesConfig.cmake"
	"#!/usr/bin/cmake

# Configuration to include MoodyCamelQueues in a CMake-based project. If you want to
# reference MoodyCamelQueues as an externally compiled static library, do this:
#
#    set(MoodyCamelQueues_DIR \"../ThirdParty/moodycamelqueues\")
#    find_package(MoodyCamelQueues REQUIRED CONFIG)
#
#    target_link_libraries(
#        MyAwesomeProject
#        PRIVATE MoodyCamelQueues::Static
#    )
#
# Alternatively, if you want to build MoodyCamelQueues together with your project,
# use the normal CMakeLists.txt with CMake's add_subdirectory() command:
#
#    add_subdirectory(
#        \"\${PROJECT_SOURCE_DIR}/../ThirdParty/moodycamelqueues\"
#        \"\${CMAKE_BINARY_DIR}/moodycamelqueues\"
#    )
#
#    target_link_libraries(
#        MyAwesomeProject
#        PRIVATE MoodyCamelQueues
#    )
#

# -------------------------------------------------------------------------------------------------

if(NOT DEFINED NUCLEX_COMPILER_TAG)
	message(
		FATAL_ERROR
		\"NUCLEX_COMPILER_TAG not defined! Include cplusplus.cmake before importing this package \\
		in order to generate a tag identifying the platform/compiler/architecture/variant!\"
	)
endif()

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}\")
	# TODO: Warn and link release build when compiling in debug mode
	# TODO: Warn and link build for older compiler version if found
	message(
		FATAL_ERROR
		\"Directory '\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}' not found. \\
		Please either build and install this project before importing it via \\
		find_package() or use this project's main CMakeFiles.txt via add_subdirectory()!\"
	)
endif()

# -------------------------------------------------------------------------------------------------

add_library(MoodyCamelQueues::Static STATIC IMPORTED)

# This may cause warnings on recent GCC versions (10.0.0+?) with LTO because GCC detects
# that the headers used during build (residing in build/) are not the same used when
# linking the library (copies resising in Include/).
#
# CMake doesn't run the install step during build, so the only way to get the headers
# in place before building would be by copying them rather than installing them.
set_target_properties(
	MoodyCamelQueues::Static PROPERTIES
	INTERFACE_INCLUDE_DIRECTORIES \"\${CMAKE_CURRENT_LIST_DIR}/Include\"
	IMPORTED_LINK_INTERFACE_LANGUAGES \"C\"
)

if(WIN32)
	set_target_properties(
		MoodyCamelQueues::Static PROPERTIES
		IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/moodycamelqueues.lib\"
	)
else()
	set_target_properties(
		MoodyCamelQueues::Static PROPERTIES
		IMPORTED_LOCATION \"\${CMAKE_CURRENT_LIST_DIR}/bin/\${NUCLEX_COMPILER_TAG}/moodycamelqueues.a\"
	)
endif()

message(STATUS \"Imported MoodyCamelQueues targets with binaries in '\${CMAKE_CURRENT_LIST_DIR}'\")"
)

# -------------------------------------------------------------------------------------------------
