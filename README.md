# Remill [![Slack Chat](http://empireslacking.herokuapp.com/badge.svg)](https://empireslacking.herokuapp.com/)

<p align="center">
     <img src="docs/images/remill_logo.png" />
</p>

Remill is a static binary translator that translates machine code instructions into [LLVM bitcode](http://llvm.org/docs/LangRef.html). It translates x86 and amd64 machine code (including AVX and AVX512) into LLVM bitcode. AArch64 support is underway.

Remill focuses on accurately lifting instructions. It is meant to be used as a library for other tools, e.g. [McSema](https://github.com/lifting-bits/mcsema).

## Build Status

[![Build Status](https://img.shields.io/github/workflow/status/lifting-bits/remill/CI/master)](https://github.com/lifting-bits/remill/actions?query=workflow%3ACI)

## Additional Documentation

 - [How to contribute](docs/CONTRIBUTING.md)
 - [How to implement the semantics of an instruction](docs/ADD_AN_INSTRUCTION.md)
 - [How instructions are lifted](docs/LIFE_OF_AN_INSTRUCTION.md)
 - [The design and architecture of Remill](docs/DESIGN.md)

## Getting Help

If you are experiencing undocumented problems with Remill then ask for help in the `#binary-lifting` channel of the [Empire Hacking Slack](https://empireslacking.herokuapp.com/).

## Supported Platforms

Remill is supported on Linux platforms and has been tested on Ubuntu 14.04, 16.04, and 18.04. Remill also works on macOS, and has experimental support for Windows.

Remill's Linux version can also be built via Docker for quicker testing.

## Dependencies

Most of Remill's dependencies can be provided by the [cxx-common](https://github.com/trailofbits/cxx-common) repository. Trail of Bits hosts downloadable, pre-built versions of cxx-common, which makes it substantially easier to get up and running with Remill. Nonetheless, the following table represents most of Remill's dependencies.

| Name | Version |
| ---- | ------- |
| [Git](https://git-scm.com/) | Latest |
| [CMake](https://cmake.org/) | 3.2+ |
| [Google Flags](https://github.com/google/glog) | Latest |
| [Google Log](https://github.com/google/glog) | Latest |
| [Google Test](https://github.com/google/googletest) | Latest |
| [LLVM](http://llvm.org/) | 3.5+ |
| [Clang](http://clang.llvm.org/) | 3.5+ |
| [Intel XED](https://software.intel.com/en-us/articles/xed-x86-encoder-decoder-software-library) | Latest |
| [Python](https://www.python.org/) | 2.7 |
| Unzip | Latest |
| [ccache](https://ccache.dev/) | Latest |

## Getting and Building the Code

### Vcpkg build

```
git submodule update --init
./vcpkg/bootstrap-vcpkg.sh
./vcpkg/vcpkg install glog gflags xed gtest

CC=clang CXX=clang++ cmake -B build -G Ninja -DCMAKE_INSTALL_PREFIX=$(pwd)/install -DCMAKE_TOOLCHAIN_FILE=$(pwd)/vcpkg/scripts/buildsystems/vcpkg.cmake -S .

cmake --build build --target test_dependencies
env CTEST_OUTPUT_ON_FAILURE=1 cmake --build build --target test
```

### Docker Build

Remill now comes with a Dockerfile for easier testing. This Dockerfile references the [cxx-common](https://github.com/trailofbits/cxx-common) container to have all pre-requisite libraries available. 

The Dockerfile allows for quick builds of multiple supported LLVM, architecture, and Linux configurations.

Quickstart (builds Remill against LLVM 8.0 on Ubuntu 18.04 for AMD64):

Clone Remill:
```shell
#Clone the repository.
git clone https://github.com/lifting-bits/remill.git
cd remill
```

Build Remill Docker container:
```shell
# do the build
docker build . -t remill:llvm800-ubuntu18.04-amd64 \
     -f Dockerfile \
     --build-arg UBUNTU_VERSION=18.04 \
     --build-arg ARCH=amd64 \
     --build-arg LLVM_VERSION=800
```

Ensure remill works:
```shell
# Decode some AMD64 instructions to LLVM
docker run --rm -it remill:llvm800-ubuntu18.04-amd64 \
     --arch amd64 --ir_out /dev/stdout --bytes c704ba01000000
     
# Decode some AArch64 instructions to LLVM
docker run --rm -it remill:llvm800-ubuntu18.04-amd64 \
     --arch aarch64 --address 0x400544 --ir_out /dev/stdout \
     --bytes FD7BBFA90000009000601891FD030091B7FFFF97E0031F2AFD7BC1A8C0035FD6
```

### On Linux

First, update aptitude and get install the baseline dependencies.

```shell
sudo apt-get update
sudo apt-get upgrade

sudo apt-get install \
     git \
     python2.7 \
     wget \
     curl \
     build-essential \
     libtinfo-dev \
     lsb-release \
     zlib1g-dev \
     ccache

# Ubuntu 14.04, 16.04
sudo apt-get install realpath
```

Next, clone the repository. This will clone the code into the `remill` directory.

```shell
git clone https://github.com/lifting-bits/remill.git
```

Next, we build Remill. This script will create another directory, `remill-build`,
in the current working directory. All remaining dependencies needed
by Remill will be built in the `remill-build` directory.

```shell
./remill/scripts/build.sh
```

Next, we can install Remill. Remill itself is a library, and so there is no real way
to try it. However, you can head on over to the [McSema](https://github.com/lifting-bits/mcsema) repository, which uses Remill for lifting instructions.

```shell
cd ./remill-build
sudo make install
```

We can also build and run Remill's test suite.

```shell
cd ./remill-build
make test_dependencies
make test
```
