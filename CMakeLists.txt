cmake_minimum_required(VERSION 3.20)

# TODO: Can this be replaced with a toolchain file?
set(CMAKE_LINK_DEPENDS_NO_SHARED TRUE)

set(CMAKE_C_COMPILER clang)

set(SDK_PATH ${CMAKE_SOURCE_DIR}/common-3.0.sdk)
set(CMAKE_LINKER ${SDK_PATH}/usr/bin/ld)
set(LIPO ${SDK_PATH}/usr/bin/lipo)

set(CMAKE_OSX_SYSROOT ${SDK_PATH}) # Does not seem to work
set(CMAKE_OSX_DEPLOYMENT_TARGET 3.0)
set(CMAKE_C_COMPILER_TARGET arm-apple-ios3.0)

set(CMAKE_C_CREATE_SHARED_LIBRARY
    "<CMAKE_LINKER> <LINK_FLAGS> -o <TARGET> <OBJECTS>") # Force CMake to use
                                                         # custom linker

# Setting OSX Flags breaks the test on Ubuntu. Claim it works as a workaround
# (since it does)
set(CMAKE_C_COMPILER_WORKS TRUE)
#

project(
  zlib
  VERSION 1.2.3
  LANGUAGES C)

set(LIB_NAME "libz.1.dylib")
set(INSTALL_PATH /usr/lib)
set(VERSION 1.2.3)
set(COMPAT_VERSION 1.0.0)

include(FetchContent)
FetchContent_Declare(
  zlib
  GIT_REPOSITORY https://github.com/apple-oss-distributions/zlib
  GIT_TAG zlib-23.0.1)
FetchContent_MakeAvailable(zlib)

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
    ${zlib_SOURCE_DIR}/zlib/zutil.c)

add_compile_options(
  -Wno-deprecated-non-prototype
  -Wno-incompatible-sysroot
  -Wno-expansion-to-defined
  -fno-stack-protector
  -isysroot
  ${SDK_PATH} # Workardound for missing
  # __stack_chk_*
)

add_compile_definitions(USE_MMAP NDEBUG)

# There are more idiomatic ways of setting some of these but stuff like
# target_link_libraries was not working for some reason
add_link_options(
  ${SDK_PATH}/usr/lib/dylib1.o
  -L${SDK_PATH}/usr/lib
  -lSystem.B
  -lgcc_s.1
  -ios_version_min
  ${CMAKE_OSX_DEPLOYMENT_TARGET}
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
  0x301f6000
  -install_name
  ${INSTALL_PATH}/${LIB_NAME})

set(arch_targets "")
# Create a target for each architecture
foreach(ARCH IN LISTS CMAKE_OSX_ARCHITECTURES)
  set(target_name libz_${ARCH})
  list(APPEND arch_targets ${target_name})
  add_library(${target_name} SHARED ${ZLIB_SOURCES})

  set_target_properties(
    ${target_name}
    PROPERTIES OUTPUT_NAME "${LIB_NAME}.${ARCH}"
               INSTALL_NAME_DIR "${INSTALL_PATH}"
               PREFIX ""
               SUFFIX ""
               POSITION_INDEPENDENTCODE OFF
               C_STANDARD 99)

  target_include_directories(${target_name} PRIVATE ${CMAKE_SOURCE_DIR}/zlib
                                                    ${SDK_PATH}/usr/include)

  target_compile_options(${target_name} PRIVATE -arch ${ARCH})

  target_link_options(${target_name} PRIVATE -arch ${ARCH})
endforeach()

set(lipo_inputs "")
foreach(ARCH IN LISTS CMAKE_OSX_ARCHITECTURES)
  list(APPEND lipo_inputs "${CMAKE_BINARY_DIR}/${LIB_NAME}.${ARCH}")
endforeach()

add_custom_target(
  ${LIB_NAME} ALL
  COMMAND ${LIPO} -create ${lipo_inputs} -output ${CMAKE_BINARY_DIR}/${LIB_NAME}
  DEPENDS ${arch_targets})
