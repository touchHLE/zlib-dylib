cmake_minimum_required(VERSION 3.20)

set(CMAKE_LINK_DEPENDS_NO_SHARED TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE OFF)
set(CMAKE_SHARED_LIBRARY_C_FLAGS "")

set(CMAKE_C_COMPILER clang)

set(SDK_PATH ${CMAKE_SOURCE_DIR}/iPhoneOS3.0-custom.sdk)

set(CMAKE_LINKER ${SDK_PATH}/usr/bin/ld)
set(LIPO ${SDK_PATH}/usr/bin/lipo)

set(CMAKE_C_CREATE_SHARED_LIBRARY "<CMAKE_LINKER> <LINK_FLAGS> -o <TARGET> <OBJECTS>") # Force CMake to use custom linker

project(zlib VERSION 1.2.3 LANGUAGES C)

set(LIB_NAME "libz.1.dylib")
set(INSTALL_PATH /usr/lib)
set(VERSION 1.2.3)
set(COMPAT_VERSION 1.0.0)

include(FetchContent)

# Auto fetch the zlib repo
FetchContent_Declare(
    zlib
    GIT_REPOSITORY https://github.com/apple-oss-distributions/zlib
    GIT_TAG zlib-23.0.1
)
FetchContent_MakeAvailable(zlib)

file(GLOB SRCS "${zlib_SOURCE_DIR}/zlib/*.c")

set(CFLAGS_COMMON
    -isysroot ${SDK_PATH}
    -pipe
    -std=gnu99
    -Wno-deprecated-non-prototype
    -fno-stack-protector
    -DUSE_MMAP
    -DNDEBUG
    --target=arm-apple-ios3.0
)

set(LDFLAGS_COMMON
    ${SDK_PATH}/usr/lib/dylib1.o
    -L${SDK_PATH}/usr/lib
    -lazy-lSystem.B
    -lazy-lgcc_s.1
    -exported_symbols_list ${zlib_SOURCE_DIR}/libz.exp
    -install_name ${INSTALL_PATH}/${LIB_NAME}
    -dead_strip
    -dylib
    -single_module
    -dead_strip_dylibs
    -prebind
    -dynamic
    -S
    -twolevel_namespace
    -twolevel_namespace_hints
    -ios_version_min 3.0
)

# Function to create a target for each architecture
function(add_arch_target ARCH)
    add_library(${ARCH}_target SHARED ${SRCS})
    
    set_target_properties(${ARCH}_target PROPERTIES 
        OUTPUT_NAME "${LIB_NAME}.${ARCH}"
        PREFIX ""
        SUFFIX ""
    )
    
    target_compile_options(${ARCH}_target PRIVATE
        ${CFLAGS_COMMON}
        -arch ${ARCH}
    )
    
    target_link_options(${ARCH}_target PRIVATE
        ${LDFLAGS_COMMON}
        -arch ${ARCH}
        -image_base 0x301f6000
    )
    
    target_include_directories(${ARCH}_target PRIVATE
        ${CMAKE_SOURCE_DIR}/zlib
        ${SDK_PATH}/usr/include
    )

endfunction()

add_arch_target(armv6)
add_arch_target(armv7)

add_custom_target(${LIB_NAME} ALL
    COMMAND ${LIPO} -create 
        ${CMAKE_BINARY_DIR}/${LIB_NAME}.armv6 
        ${CMAKE_BINARY_DIR}/${LIB_NAME}.armv7 
        -output ${CMAKE_BINARY_DIR}/${LIB_NAME}
    DEPENDS armv6_target armv7_target
)