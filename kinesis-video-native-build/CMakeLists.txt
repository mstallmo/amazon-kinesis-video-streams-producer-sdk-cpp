cmake_minimum_required(VERSION 3.2)

project(start)

set(EXEC_NAME start)

set(CMAKE_CXX_STANDARD 14)

if(DEFINED ENV{KINESIS_VIDEO_PIC_SRC})
    set(KINESIS_VIDEO_PIC_SRC $ENV{KINESIS_VIDEO_PIC_SRC})
else()
    set(KINESIS_VIDEO_PIC_SRC "../kinesis-video-pic")
endif()

if(DEFINED ENV{KINESIS_VIDEO_OPEN_SOURCE_SRC})
    set(KINESIS_VIDEO_OPEN_SOURCE_SRC $ENV{KINESIS_VIDEO_OPEN_SOURCE_SRC})
else()
    set(KINESIS_VIDEO_OPEN_SOURCE_SRC "${PROJECT_BINARY_DIR}/downloads")
endif()

message(STATUS "System name is ${CMAKE_SYSTEM_NAME}")

STRING(TOLOWER "${CMAKE_SYSTEM_NAME}" CMAKE_SYSTEM_NAME_LOWER_CASE)

message(STATUS "Kinesis Video PIC path is ${KINESIS_VIDEO_PIC_SRC}")
message(STATUS "Kinesis Video Open Source path is ${KINESIS_VIDEO_OPEN_SOURCE_SRC}")

message(STATUS "Configuration ${CMAKE_GENERATOR}")

if(CMAKE_SIZEOF_VOID_P STREQUAL 4)
    message(STATUS "Bitness 32 bits")
	set(KINESIS_VIDEO_BUILD_BITNESS "x86")
	set(KINESIS_VIDEO_BUILD_BITNESS_TYPE "Win32")
	set(KINESIS_VIDEO_BUILD_BITNESS_NAME "x86")
elseif(CMAKE_SIZEOF_VOID_P STREQUAL 8)
	message(STATUS "Bitness 64 bits")
	set(KINESIS_VIDEO_BUILD_BITNESS "x64")
	set(KINESIS_VIDEO_BUILD_BITNESS_TYPE "x64")
	set(KINESIS_VIDEO_BUILD_BITNESS_NAME "x86_64")
else()
	message(FATAL_ERROR "Unknown bitness")
endif()

STRING(TOLOWER "${CMAKE_BUILD_TYPE}" CMAKE_BUILD_TYPE_LOWER)

# Uncomment below line for debug heap
#add_definitions(-DHEAP_DEBUG)

# Uncomment below line for very verbose logging
#add_definitions(-DLOG_STREAMING)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
endif()

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads)
set(EXE_LIBRARIES Threads::Threads ${CMAKE_DL_LIBS})

if(NOT ${CMAKE_SYSTEM_NAME} MATCHES "Darwin" AND NOT ${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  set(EXE_LIBRARIES ${EXE_LIBRARIES} rt)
endif()

message(STATUS "exe libraries ${EXE_LIBRARIES}")

set(LIBRARY_PATH ${KINESIS_VIDEO_OPEN_SOURCE_SRC}/local)

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--export-all-symbols")
    set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -Wl,--export-all-symbols")
  endif()
  set(LIBRARY_PATH $ENV{MINGW_PREFIX})
  add_definitions(-DCURL_STATICLIB -DWIN32_LEAN_AND_MEAN -DVC_EXTRALEAN -D_HAS_ITERATOR_DEBUGGING=0)
endif()

if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS -D_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING)
endif()

if(NOT BUILD_JNI_ONLY AND (NOT ${CMAKE_SYSTEM_NAME} MATCHES "Windows" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU"))
  find_package(PkgConfig REQUIRED)

  pkg_check_modules (LOG4CPLUS    REQUIRED   log4cplus=1.2.0)

  if (NOT NO_UNIT_TEST)
    find_library(GTEST NAMES gtest REQUIRED HINTS ${LIBRARY_PATH} PATH_SUFFIXES lib)
    find_library(GTEST_MAIN NAMES gtest_main REQUIRED HINTS ${LIBRARY_PATH} PATH_SUFFIXES lib)
  endif()

  set(OPEN_SRC_INCLUDE_DIRS
          ${LOG4CPLUS_INCLUDE_DIRS}
          ${GTEST_INCLUDE_DIRS}
          ${GTEST_MAIN_INCLUDE_DIRS})

  set(OPEN_SRC_LIBRARY_DIRS
          ${LOG4CPLUS_LIBRARY_DIRS}
          ${GTEST_LIBRARY_DIRS}
          ${GTEST_MAIN_LIBRARY_DIRS})

  set(PRODUCER_LIBRARIES ${LOG4CPLUS_LIBRARIES})
endif()

set(PIC_SOURCE_FILES
        ${KINESIS_VIDEO_PIC_SRC}/src/client/include/com/amazonaws/kinesis/video/client/Include.h
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/AuthIntegration.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/Callbacks.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/Client.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/ClientEvent.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/ClientState.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/Include_i.h
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/InputValidator.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/InputValidator.h
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/State.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/State.h
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/AckParser.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/Stream.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/Stream.h
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/StreamEvent.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/src/StreamState.cpp

        ${KINESIS_VIDEO_PIC_SRC}/src/common/include/com/amazonaws/kinesis/video/common/CommonDefs.h
        ${KINESIS_VIDEO_PIC_SRC}/src/common/include/com/amazonaws/kinesis/video/common/PlatformUtils.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/include/com/amazonaws/kinesis/video/heap/Include.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/AivHeap.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/AivHeap.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/Common.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/Common.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/Heap.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/HybridHeap.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/HybridHeap.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/Include_i.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/SystemHeap.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/src/SystemHeap.h

        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/include/com/amazonaws/kinesis/video/mkvgen/Include.h
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/src/Include_i.h
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/src/MkvGenerator.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/src/MkvStatics.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/src/NalAdapter.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/src/SpsParser.cpp

        ${KINESIS_VIDEO_PIC_SRC}/src/trace/include/com/amazonaws/kinesis/video/trace/Include.h
        ${KINESIS_VIDEO_PIC_SRC}/src/trace/src/Include_i.h
        ${KINESIS_VIDEO_PIC_SRC}/src/trace/src/TraceProfiler.cpp

        ${KINESIS_VIDEO_PIC_SRC}/src/utils/include/com/amazonaws/kinesis/video/utils/Include.h
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Allocators.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Base64.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/BitField.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/BitReader.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Directory.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/dlfcn_win_stub.h
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/DoubleLinkedList.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/DynamicLibrary.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Endianness.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/FileIo.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/HashTable.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Hex.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Include_i.h
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Mutex.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/SingleLinkedList.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/StackQueue.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/String.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Thread.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Time.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/src/Logger.cpp

        ${KINESIS_VIDEO_PIC_SRC}/src/view/include/com/amazonaws/kinesis/video/view/Include.h
        ${KINESIS_VIDEO_PIC_SRC}/src/view/src/ContentView.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/view/src/Include_i.h)

set(TST_PRODUCER_SOURCE_FILES
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/ClientApiFunctionalityTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/ClientApiTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamPutGetTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/ClientTestFixture.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/ClientTestFixture.h
        #${KINESIS_VIDEO_PIC_SRC}/src/client/tst/main.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamApiFunctionalityTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamApiServiceCallsTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamApiTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamDeviceTagsTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamParallelTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamStateTransitionsTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/client/tst/StreamTokenRotationTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/tst/HeapApiFunctionalityTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/tst/HeapApiTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/tst/HeapTestFixture.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/tst/HeapTestFixture.h
        ${KINESIS_VIDEO_PIC_SRC}/src/heap/tst/HybridHeapTest.cpp
        #${KINESIS_VIDEO_PIC_SRC}/src/heap/tst/main.cpp
        #${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/main.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/MkvgenApiFunctionalityTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/MkvgenApiTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/MkvgenTestFixture.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/MkvgenTestFixture.h
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/AnnexBNalAdapterTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/AnnexBCpdNalAdapterTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/AvccNalAdapterTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst/SpsParserTest.cpp
        #${KINESIS_VIDEO_PIC_SRC}/src/trace/tst/main.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/trace/tst/TraceApiFunctionalityTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/trace/tst/TraceApiTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/trace/tst/TraceTestFixture.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/trace/tst/TraceTestFixture.h
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/BitField.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/BitReader.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/Directory.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/Thread.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/DoubleLinkedList.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/HashTable.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/IntegerToString.cpp
        #${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/main.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/SingleLinkedList.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/StackQueue.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/StringToInteger.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/utils/tst/UtilTestFixture.cpp
	#${KINESIS_VIDEO_PIC_SRC}/src/view/tst/main.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/view/tst/ViewApiFunctionalityTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/view/tst/ViewApiTest.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/view/tst/ViewTestFixture.cpp
        ${KINESIS_VIDEO_PIC_SRC}/src/view/tst/ViewTestFixture.h)

include_directories(${KINESIS_VIDEO_PIC_SRC})
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/client/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/client/include/com/amazonaws/kinesis/video/client)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/client/src)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/client/tst)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/common/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/common/include/com/amazonaws/kinesis/video/common)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/heap/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/heap/include/com/amazonaws/kinesis/video/heap)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/heap/src)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/heap/tst)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/trace/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/include/com/amazonaws/kinesis/video/mkvgen)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/src)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/mkvgen/tst)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/view/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/view/include/com/amazonaws/kinesis/video/view)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/view/src)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/view/tst)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/utils/include)
include_directories(${KINESIS_VIDEO_PIC_SRC}/src/utils/src)
include_directories(${OPEN_SRC_INCLUDE_DIRS})

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  include_directories("${JAVA_HOME}/include/win32")
  include_directories("${KINESIS_VIDEO_OPEN_SOURCE_SRC}/googletest-release-1.8.0/googletest/include")
  include_directories("${KINESIS_VIDEO_OPEN_SOURCE_SRC}/log4cplus-1.2.1/include")
endif()

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows" AND NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  macro(FORCE_MSVS_CRT_TYPE)
    if(MSVC)
      set(variables
        CMAKE_C_FLAGS_DEBUG
        CMAKE_C_FLAGS_MINSIZEREL
        CMAKE_C_FLAGS_RELEASE
        CMAKE_C_FLAGS_RELWITHDEBINFO
        CMAKE_CXX_FLAGS_DEBUG
        CMAKE_CXX_FLAGS_MINSIZEREL
        CMAKE_CXX_FLAGS_RELEASE
        CMAKE_CXX_FLAGS_RELWITHDEBINFO
      )

      if(${MSVS_CRT_TYPE} STREQUAL "" OR ${MSVS_CRT_TYPE} STREQUAL "static")
        message(STATUS "Using static CRT")
        foreach(variable ${variables})
          if(${variable} MATCHES "/MD")
	    if("Debug" STREQUAL "${CMAKE_BUILD_TYPE}" OR "Debug" STREQUAL "${CMAKE_CONFIGURATION_TYPES}")
              string(REGEX REPLACE "/MD" "/MTd" ${variable} "${${variable}}")
            else()
              string(REGEX REPLACE "/MD" "/MT" ${variable} "${${variable}}")
            endif()
          endif()
        endforeach()
      else()
        message(STATUS "Using dynamic CRT")
        foreach(variable ${variables})
          if(${variable} MATCHES "/MT")
            if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
              string(REGEX REPLACE "/MT" "/MDd" ${variable} "${${variable}}")
            else()
              string(REGEX REPLACE "/MT" "/MD" ${variable} "${${variable}}")
            endif()
          endif()
        endforeach()
      endif()
    endif()
  endmacro()

  set(MSVS_CRT_TYPE "static")
  FORCE_MSVS_CRT_TYPE()

  add_definitions(-DCURL_STATICLIB -DWIN32_LEAN_AND_MEAN -DVC_EXTRALEAN -D_HAS_ITERATOR_DEBUGGING=0)

  link_directories(${KINESIS_VIDEO_OPEN_SOURCE_SRC}/local/lib)

  link_directories(${OPEN_SRC_LIBRARY_DIRS})
  add_library(producer STATIC ${PRODUCER_SOURCE_FILES})
  add_executable(start ${TST_PRODUCER_SOURCE_FILES})

  target_link_libraries(start
          producer)

  target_link_libraries(producer
          ws2_32
          crypt32
          Wldap32
          Normaliz
          libcurl_a)

  target_link_libraries(producer
          optimized ${KINESIS_VIDEO_OPEN_SOURCE_SRC}/log4cplus-1.2.1/msvc10/${KINESIS_VIDEO_BUILD_BITNESS_TYPE}/bin.Release/log4cplusS.lib
          optimized ${KINESIS_VIDEO_OPEN_SOURCE_SRC}/googletest-release-1.8.0/googletest/msvc/${KINESIS_VIDEO_BUILD_BITNESS_TYPE}/Release/gtest.lib)
else()

  link_directories(${OPEN_SRC_LIBRARY_DIRS})

  add_library(producer STATIC ${PIC_SOURCE_FILES})
  target_link_libraries(producer
          ${PRODUCER_LIBRARIES}
          ${GTEST}
          ${GTEST_MAIN})

  add_executable(start ${TST_PRODUCER_SOURCE_FILES})
  target_link_libraries(start
          producer
          Threads::Threads
          ${CMAKE_DL_LIBS}
          ${LOG4CPLUS_LIBRARIES})
endif()
