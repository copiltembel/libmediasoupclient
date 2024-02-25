cmake_minimum_required(VERSION 3.14)

project(mediasoupclient LANGUAGES CXX)

include(FetchContent)

# Set version number.
set(mediasoupclient_VERSION_MAJOR 3)
set(mediasoupclient_VERSION_MINOR 4)
set(mediasoupclient_VERSION_PATCH 2)

# Configure a header file to pass the version.
configure_file (
	"${PROJECT_SOURCE_DIR}/version.hpp.in"
	"${PROJECT_SOURCE_DIR}/include/version.hpp"
	NEWLINE_STYLE LF
)

# C++ standard requirements.
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Project options.
option(MEDIASOUPCLIENT_BUILD_TESTS "Build unit tests" OFF)
option(MEDIASOUPCLIENT_LOG_TRACE   "Enable MSC_LOG_TRACE (See Logger.hpp)" OFF)
option(MEDIASOUPCLIENT_LOG_DEV     "Enable MSC_LOG_DEV (See Logger.hpp)" OFF)
option(USE_CUSTOM_LIBC_FROM_LIBWEBRTC "Use custom libc++ from libwebrtc - this will disable linking against the (default) system libc++" OFF)

# Project configuration.
set(LIBWEBRTC_INCLUDE_PATH "" CACHE STRING "libwebrtc include path")
set(LIBWEBRTC_BINARY_PATH "" CACHE STRING "libwebrtc binary path")

if(NOT LIBWEBRTC_INCLUDE_PATH)
	message(FATAL_ERROR "LIBWEBRTC_INCLUDE_PATH not provided")
endif()

if(NOT LIBWEBRTC_BINARY_PATH)
	message(FATAL_ERROR "LIBWEBRTC_BINARY_PATH not provided")
endif()

message("\n=========== libmediasoupclient Build Configuration ===========\n")
message(STATUS "MEDIASOUPCLIENT_BUILD_TESTS : " ${MEDIASOUPCLIENT_BUILD_TESTS})
message(STATUS "MEDIASOUPCLIENT_LOG_TRACE   : " ${MEDIASOUPCLIENT_LOG_TRACE})
message(STATUS "MEDIASOUPCLIENT_LOG_DEV     : " ${MEDIASOUPCLIENT_LOG_DEV})
message(STATUS "LIBWEBRTC_INCLUDE_PATH      : " ${LIBWEBRTC_INCLUDE_PATH})
message(STATUS "LIBWEBRTC_BINARY_PATH       : " ${LIBWEBRTC_BINARY_PATH})
message(STATUS "USE_CUSTOM_LIBC_FROM_LIBWEBRTC:       : " ${USE_CUSTOM_LIBC_FROM_LIBWEBRTC})
message("")

# GCC >= 4.9 is required due to std::regex use.
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
		message(FATAL_ERROR "GCC version must be at least 4.9!")
	endif()
endif()

if (${MEDIASOUPCLIENT_BUILD_TESTS})
	add_subdirectory(test)
endif()

set(HEADER_FILES
	include/mediasoupclient.hpp
)

set(
	SOURCE_FILES
	src/Consumer.cpp
	src/DataConsumer.cpp
	src/DataProducer.cpp
	src/Device.cpp
	src/Handler.cpp
	src/Logger.cpp
	src/PeerConnection.cpp
	src/Producer.cpp
	src/Transport.cpp
	src/mediasoupclient.cpp
	src/ortc.cpp
	src/scalabilityMode.cpp
	src/sdp/MediaSection.cpp
	src/sdp/RemoteSdp.cpp
	src/sdp/Utils.cpp
	include/Consumer.hpp
	include/Device.hpp
	include/Handler.hpp
	include/Logger.hpp
	include/MediaSoupClientErrors.hpp
	include/PeerConnection.hpp
	include/Producer.hpp
	include/Transport.hpp
	include/mediasoupclient.hpp
	include/ortc.hpp
	include/scalabilityMode.hpp
	include/sdp/MediaSection.hpp
	include/sdp/RemoteSdp.hpp
	include/sdp/Utils.hpp
)

if (${USE_CUSTOM_LIBC_FROM_LIBWEBRTC}) 
	set(CUSTOM_LIBC_OBJ_FILES
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/algorithm.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/any.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/atomic.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/barrier.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/bind.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/charconv.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/chrono.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/condition_variable_destructor.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/condition_variable.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/d2fixed.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/d2s.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/directory_entry.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/directory_iterator.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/error_category.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/exception.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/f2s.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/filesystem_clock.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/filesystem_error.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/functional.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/future.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/hash.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/ios.instantiations.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/ios.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/iostream.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/legacy_pointer_safety.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/locale.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/memory.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/mutex_destructor.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/mutex.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/new_handler.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/new_helpers.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/new.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/operations.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/optional.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/path.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/random.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/random_shuffle.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/regex.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/shared_mutex.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/stdexcept.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/string.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/strstream.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/system_error.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/thread.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/typeinfo.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/valarray.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/variant.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/vector.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++/libc++/verbose_abort.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/abort_message.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_aux_runtime.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_default_handlers.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_demangle.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_exception.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_exception_storage.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_guard.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_handlers.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_personality.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_thread_atexit.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_vector.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/cxa_virtual.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/fallback_malloc.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/private_typeinfo.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/stdlib_exception.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/stdlib_stdexcept.o
		${LIBWEBRTC_BINARY_PATH}/buildtools/third_party/libc++abi/libc++abi/stdlib_typeinfo.o	
	)

	set(CMAKE_CXX_IMPLICIT_LINK_LIBRARIES "")
	set(CMAKE_CXX_IMPLICIT_LINK_DIRECTORIES "")

endif()

# Create target.
add_library(${PROJECT_NAME} STATIC ${SOURCE_FILES})

if(${MEDIASOUPCLIENT_LOG_TRACE})
	target_compile_definitions(${PROJECT_NAME}
		PRIVATE MSC_LOG_TRACE=1
	)
endif()

if(${MEDIASOUPCLIENT_LOG_DEV})
	target_compile_definitions(${PROJECT_NAME}
		PRIVATE MSC_LOG_DEV=1
	)
endif()

# Source Dependencies.

message(STATUS "\nFetching libsdptransform...\n")
FetchContent_Declare(
	libsdptransform
  	GIT_REPOSITORY https://github.com/ibc/libsdptransform.git
  	GIT_TAG 1.2.10
)
FetchContent_MakeAvailable(libsdptransform)

# Add some compile flags to our source files.
if(MSVC)
	set_source_files_properties(${SOURCE_FILES}
		PROPERTIES COMPILE_FLAGS "/W3")
else()
	set_source_files_properties(${SOURCE_FILES}
		PROPERTIES COMPILE_FLAGS -Wall -Wextra -Wpedantic)
endif()

# Private (implementation) header files.
target_include_directories(${PROJECT_NAME} PRIVATE ${PROJECT_SOURCE_DIR}/include)

# Public (interface) headers from dependencies.
target_include_directories(${PROJECT_NAME} PUBLIC
    ${libsdptransform_SOURCE_DIR}/include
	"${LIBWEBRTC_INCLUDE_PATH}"
	"${LIBWEBRTC_INCLUDE_PATH}/third_party/abseil-cpp"
)

# Public (interface) dependencies.
target_link_libraries(${PROJECT_NAME} PUBLIC
	${CUSTOM_LIBC_OBJ_FILES}
	sdptransform
	${LIBWEBRTC_BINARY_PATH}/libwebrtc${CMAKE_STATIC_LIBRARY_SUFFIX}
)

# Compile definitions for libwebrtc.
target_compile_definitions(${PROJECT_NAME} PUBLIC
	$<$<NOT:$<PLATFORM_ID:Windows>>:WEBRTC_POSIX>
	$<$<PLATFORM_ID:Windows>:WEBRTC_WIN>
	$<$<PLATFORM_ID:Windows>:NOMINMAX>
	$<$<PLATFORM_ID:Windows>:WIN32_LEAN_AND_MEAN>
	$<$<PLATFORM_ID:Darwin>:WEBRTC_MAC>
)

install(TARGETS mediasoupclient DESTINATION lib)
install(FILES ${HEADER_FILES} DESTINATION include/mediasoupclient)
