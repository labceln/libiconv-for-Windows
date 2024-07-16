cmake_minimum_required(VERSION 3.15)  # CMAKE_MSVC_RUNTIME_LIBRARY requires CMake 3.15 or later

project(libiconv VERSION 1.17)

# Set C standard
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /GF")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /GF")

# Set MSVC runtime library
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# Define source files
set(SOURCES
    libiconv/libcharset/lib/localcharset.c
    libiconv/libcharset/lib/relocatable-stub.c
    libiconv/lib/iconv.c
    libiconv/lib/relocatable.c
)

# Define header files
set(HEADERS
    include/iconv.h
    libiconv/include/iconv.h
    libiconv/libcharset/config.h
    libiconv/libcharset/include/libcharset.h
    libiconv/libcharset/include/localcharset.h
)

# Define resource files
set(RESOURCES
    res/libiconv.rc
)

# Include directories
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/libiconv/libcharset/include
    ${CMAKE_CURRENT_SOURCE_DIR}/libiconv/include
    ${CMAKE_CURRENT_SOURCE_DIR}/libiconv
)

# Option to choose between static and shared library
option(BUILD_SHARED_LIBS "Build shared library" OFF)

# Define library target
add_library(libiconv ${SOURCES} ${HEADERS} ${RESOURCES})

# Set target properties
set_target_properties(libiconv PROPERTIES
    OUTPUT_NAME "libiconv"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
)

# Define compile definitions
target_compile_definitions(libiconv PRIVATE
    WIN32
    _CRT_SECURE_NO_WARNINGS
    BUILDING_LIBICONV
    BUILDING_LIBCHARSET
    UNICODE 
    _UNICODE
)

if(BUILD_SHARED_LIBS)
    target_compile_definitions(libiconv PRIVATE LIBICONV_EXPORTS)
else()
    target_compile_definitions(libiconv PRIVATE USING_STATIC_LIBICONV)
endif()

# Set compile options
target_compile_options(libiconv PRIVATE
    /W3
    /wd4311 /wd4244 /wd4117 /wd4090 /wd4267 /wd4273 /wd4018
    $<$<CONFIG:>:/MT>
    $<$<CONFIG:Debug>:/MTd>
)

# Set link options
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set_target_properties(libiconv PROPERTIES LINK_FLAGS "/MACHINE:x64")
else()
    set_target_properties(libiconv PROPERTIES LINK_FLAGS "/MACHINE:x86")
endif()

# Configure for different build types
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(libiconv PRIVATE _DEBUG)
    set_target_properties(libiconv PROPERTIES DEBUG_POSTFIX "D")
else()
    target_compile_definitions(libiconv PRIVATE NDEBUG)
endif()

# Rename static library
if(NOT BUILD_SHARED_LIBS)
    set_target_properties(libiconv PROPERTIES OUTPUT_NAME "libiconvStatic")
endif()
