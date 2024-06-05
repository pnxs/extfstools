cmake_minimum_required(VERSION 3.10)
project(extfstools)
set(CMAKE_CXX_STANDARD 17)

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    set(LINUX TRUE)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    set(DARWIN TRUE)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
    set(FREEBSD TRUE)
endif()

# checking for if we are called in the correct way:
#  with a -B argument.  and without a cache file in the source directory.
if (CMAKE_CACHEFILE_DIR STREQUAL "${CMAKE_SOURCE_DIR}")
    message(FATAL_ERROR "\nUnexpected CMakeCache.txt file in the source directory. Please remove it.")
    return()
endif()

if (EXISTS ${CMAKE_BINARY_DIR}/CMakeLists.txt)
    message(FATAL_ERROR "\nRun cmake with an explicit -B buildpath")
    return()
endif()

if(MSVC)
    # /MP = multithreaded build
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
    # /utf-8 = utf8 source and execution
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /utf-8")
endif()

if (WIN32)
    add_definitions(-DNOMINMAX -DNOGDI)
endif()


if (WIN32)
    include(FetchContent)
    FetchContent_Populate(boost
        URL      https://dl.bintray.com/boostorg/release/1.75.0/source/boost_1_75_0.tar.bz2
    )
    # todo: check fetch result.
    add_library(BoostHeaders INTERFACE)
    target_include_directories(BoostHeaders INTERFACE ${CMAKE_BINARY_DIR}/boost-src)
    target_compile_definitions(BoostHeaders INTERFACE BOOST_ALL_NO_LIB)
    set(MMAPLIB BoostHeaders)
else()
    #find_package(cpputils)
    if (cpputils_FOUND)
        message(NOTICE "uisng cpputils")
        set(MMAPLIB cpputils)
        add_definitions(-DUSE_CPPUTILS)
    endif()
endif()

add_executable(ext2rd ext2rd.cpp vectorutils.cpp stringutils.cpp)

target_link_libraries(ext2rd ${MMAPLIB})
target_include_directories(ext2rd PRIVATE ${CMAKE_SOURCE_DIR})
target_link_options(ext2rd PRIVATE "-static")
target_compile_definitions(ext2rd PRIVATE -DUSE_CPPUTILS)

install(TARGETS ext2rd DESTINATION bin)
