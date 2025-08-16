# Building zlib dylib 

## Prerequisites
Download and extract the common-3.0.sdk:
- Download the latest sdk from: <https://github.com/touchHLE/common-3.0-sdk/releases/latest>
- Extract in project root

## Build Dylib

`cmake -DCMAKE_TOOLCHAIN_FILE="common-3.0.sdk/cmake/Toolchain/common-3.0.cmake" -S . -B build`

`cmake --build build`

## Licensing 

This repository contains components under multiple licenses:

zlib dylib binary
- Licensed under zlib License
- Full license text in `LICENSE.zlib`

All other repo content
- Licensed under Mozilla Public License 2.0
- Full license text in `LICENSE`