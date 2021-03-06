cmake_minimum_required(VERSION 2.8)
project(azmq)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
   set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++11")
   set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -stdlib=libc++")
elseif (NOT MSVC)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++11")
endif()

if (${CMAKE_CXX_PLATFORM_ID} STREQUAL "Windows")
    # Build for Windows Vista / Windows Server 2008 and later
    add_definitions(-D_WIN32_WINNT=0x0600)

    add_definitions(-DNOMINMAX -DWIN32_LEAN_AND_MEAN -D_WINSOCK_DEPRECATED_NO_WARNINGS -D_SCL_SECURE_NO_WARNINGS)

    if (NOT DEFINED Boost_USE_STATIC_LIBS)
        set(Boost_USE_STATIC_LIBS ON) # On Windows, Boost is compiled as static libraries by default
    endif()
endif()

if (ZMQ_STATIC)
    add_definitions(-D ZMQ_STATIC)
endif()

set(ADDITIONAL_LIBS "")

if(USE_LIBCXX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -lc++abi")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -lc++abi")
    set(ADDITIONAL_LIBS "pthread")
endif()

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/config")

find_package(Boost 1.54 COMPONENTS system log date_time thread chrono regex random REQUIRED)
find_package(ZeroMQ 4.0 REQUIRED)

set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package(Threads REQUIRED)

if (NOT CMAKE_THREAD_LIBS_INIT)
    set(CMAKE_THREAD_LIBS_INIT "")
endif()

include_directories(${Boost_INCLUDE_DIRS}
                    ${ZeroMQ_INCLUDE_DIRS}
                    ${PROJECT_SOURCE_DIR})

enable_testing()

macro(add_catch_test name)
    if (TEST_REPORT_FORMAT)
        add_test(NAME ${name} COMMAND ${name} -r ${TEST_REPORT_FORMAT} -o "${name}.test_out.xml")
    else()
        add_test(NAME ${name} COMMAND ${name})
    endif()
endmacro()

add_subdirectory(test)
add_subdirectory(doc)

install(DIRECTORY ${PROJECT_SOURCE_DIR}/azmq
        DESTINATION include)
