cmake_minimum_required(VERSION 3.20)

set(CMAKE_ARCHITECTURES "armv6;armv7")

project(zlib VERSION 1.2.3 LANGUAGES C)

set(LIB_NAME "libz.1.2.3.dylib")
set(INSTALL_PATH /usr/lib)
set(VERSION 1.2.3)
set(COMPAT_VERSION 1.0.0)

set(IMAGE_BASE 0x301f6000)

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/zlib-src")
    message(
        FATAL_ERROR
        "zlib source is missing! Run 'git submodule update --init' to sync"
    )
endif()

set(zlib_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/zlib-src")

set(ZLIB_SOURCES
    ${zlib_SOURCE_DIR}/zlib/adler32.c
    ${zlib_SOURCE_DIR}/zlib/compress.c
    ${zlib_SOURCE_DIR}/zlib/crc32.c
    ${zlib_SOURCE_DIR}/zlib/deflate.c
    ${zlib_SOURCE_DIR}/zlib/gzio.c
    ${zlib_SOURCE_DIR}/zlib/infback.c
    ${zlib_SOURCE_DIR}/zlib/inffast.c
    ${zlib_SOURCE_DIR}/zlib/inflate.c
    ${zlib_SOURCE_DIR}/zlib/inftrees.c
    ${zlib_SOURCE_DIR}/zlib/trees.c
    ${zlib_SOURCE_DIR}/zlib/uncompr.c
    ${zlib_SOURCE_DIR}/zlib/zutil.c
)

add_library(${LIB_NAME} SHARED ${ZLIB_SOURCES})

set_target_properties(${LIB_NAME} PROPERTIES
    OUTPUT_NAME "z.${VERSION}"
    VERSION ${VERSION}
    MACHO_COMPATIBILITY_VERSION ${COMPAT_VERSION}
    INSTALL_NAME_DIR "${INSTALL_PATH}"
    POSITION_INDEPENDENT_CODE OFF
    C_STANDARD 99
)

target_compile_options(${LIB_NAME} 
    PRIVATE
        -Wno-deprecated-non-prototype
        -Wno-expansion-to-defined
        -fno-stack-protector
)

target_compile_definitions(${LIB_NAME} 
    PRIVATE
        USE_MMAP
        NDEBUG
)

target_link_options(${LIB_NAME}
    PRIVATE
        -exported_symbols_list
        ${zlib_SOURCE_DIR}/libz.exp
        -dead_strip
        -dead_strip_dylibs
        -dylib
        -single_module
        -prebind
        -dynamic
        -twolevel_namespace
        -twolevel_namespace_hints
        -image_base
        ${IMAGE_BASE}
        -install_name
        ${INSTALL_PATH}/${LIB_NAME}
)