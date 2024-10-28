# CMake

> **WARNING**: The CMake scripts do not have feature parity with the SCons ones at this stage and are still a work in progress. There are a number of people who have been working on alternative cmake solutions that are frequently referenced in the discord chats: [Ivan's cmake-rewrite branch](https://github.com/IvanInventor/godot-cpp/tree/cmake-rewrite) | [Vorlac's godot-roguelite Project](https://github.com/vorlac/godot-roguelite)

Compiling godot-cpp independently of an extension project is mainly for godot-cpp developers, package maintainers, and CI/CD. Look to the [godot-cpp-template](https://github.com/godotengine/godot-cpp-template) for a practical example on how to consume the godot-cpp library as part of a Godot extension.

[Configuration examples](#Examples) are listed at the bottom of the page.

## Clone the git repository

```shell
> git clone https://github.com/godotengine/godot-cpp.git
Cloning into 'godot-cpp'...
...
> cd godot-cpp
```

## Out-of-tree build directory
Create a build directory for cmake to put caches and build artifacts in and change directory to it. This is typically as a sub-directory of the project root but can be outside the source tree. This is so that generated files do not clutter up the source tree.
```shell
> mkdir cmake-build
> cd cmake-build
```

## Configure the build
Cmake isn't a build tool, it's a build tool generator, which means it generates the build scripts/files that will end up compiling your code.
To see the list of generators run `cmake --help`

The current working directory is the build directory that was created in the previous step.

Configure and generate Ninja build files.
Review [build-configurations](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#build-configurations) for more information.

```shell
> cmake ../ -G "Ninja"
```

To list the available options cmake use the `-L[AH]` option. `A` is for advanced, and `H` is for help strings.
```shell
> cmake ../ -LH
```

Specify options on the command line

Review [setting-build-variables](https://cmake.org/cmake/help/latest/guide/user-interaction/index.html#setting-build-variables) for more information.

```shell
> cmake ../ -DGODOT_USE_HOT_RELOAD:BOOL=ON -DGODOT_PRECISION:STRING=double -DCMAKE_BUILD_TYPE:STRING=Debug
```

### A non-exhaustive list of options
```
// Path to a custom GDExtension API JSON file (takes precedence over `GODOT_GDEXTENSION_DIR`) ( /path/to/custom_api_file )
`GODOT_CUSTOM_API_FILE:FILEPATH=`

// Force disabling exception handling code (ON|OFF)
GODOT_DISABLE_EXCEPTIONS:BOOL=ON

// Path to a custom directory containing GDExtension interface header and API JSON file ( /path/to/gdextension_dir )
GODOT_GDEXTENSION_DIR:PATH=gdextension

// Generate a template version of the Node class's get_node. (ON|OFF)
GODOT_GENERATE_TEMPLATE_GET_NODE:BOOL=ON

// Set the floating-point precision level (single|double)
GODOT_PRECISION:STRING=single

// Symbols visibility on GNU platforms. Use 'auto' to apply the default value. (auto|visible|hidden)
GODOT_SYMBOL_VISIBILITY:STRING=hidden

// Expose headers as SYSTEM.
GODOT_SYSTEM_HEADERS:BOOL=ON

// Enable the extra accounting required to support hot reload. (ON|OFF)
GODOT_USE_HOT_RELOAD:BOOL=

// Treat warnings as errors
GODOT_WARNING_AS_ERROR:BOOL=OFF
```

## Building

Building the default `all` target, this will build `editor`, `template_debug` and `template_release` targets.
```shell
> cmake --build .
```

To build only a single target, specify it like so:

```shell
> cmake --build . -t template_release
```

When using multi-config generators like `Ninja Multi-Config`, `Visual Studio *` or `Xcode` the build configuration needs to be specified at build time. Build in Release mode unless you need debug symbols.

```shell
> cmake --build . -t template_release --config Release
```

## Examples

#### Building using Microsoft Visual Studio
So long as cmake is installed from the cmake [downloads page](https://cmake.org/download/) and in the PATH, and Microsoft Visual Studio is installed with c++ support, cmake will detect the msvc compiler.

Assuming the current working directory is the godot-cpp project root:
```shell
> mkdir build-msvc
> cd build-msvc
> cmake ../
> cmake --build . -t godot-cpp-test --config Release
```

#### Building godot-cpp-test with debug symbols using msys2/clang64 and "Ninja" generator
Assumes the ming-w64-clang-x86_64-toolchain is installed

Using the msys2/clang64 shell
```shell
> mkdir build-clang
> cd build-clang
> cmake ../ -G"Ninja" -DCMAKE_BUILD_TYPE:STRING=Debug
> cmake --build . -t godot-cpp-test
```

#### Building godot-cpp-test with debug symbols using msys2/clang64 and "Ninja Multi-Config" generator
Assumes the ming-w64-clang-x86_64-toolchain is installed

Using the msys2/clang64 shell
```shell
> mkdir build-clang
> cd build-clang
> cmake ../ -G"Ninja Multi-Config"
> cmake --build . -t godot-cpp-test --config Debug
```


### Emscripten to web-assembly
I've only tested this on windows so far.

I cloned, installed, and activating the latest Emscripten tools( for me it was 3.1.69) to c:\emsdk

From a terminal running the c:\emsdk\emcmdprompt.bat puts me in a cmdprompt context which I dislike, so after that I run pwsh to get my powershell 7.4.5 context back.

using the emcmake.bat command adds the emscripten toolchain to the cmake command

```shell
> C:\emsdk\emcmdprompt.bat
> pwsh
> cd <godot-cpp source folder>
> mkdir build-wasm32
> cd build-wasm32
> emcmake.bat cmake ../
> cmake --build . --verbose -t template_release
```

### Android Cross Compile from Windows
From what I can tell, there are two directions you can go

Use the `CMAKE_ANDROID_*` variables specified on the commandline or in your own toolchain file as listed in the [cmake-toolchains](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html#cross-compiling-for-android-with-the-ndk) documentation.

Or use the `$ANDROID_HOME/ndk/<version>/build/cmake/android.toolchain.cmake` toolchain and make changes using the `ANDROID_*` variables listed there. Where `<version>` is whatever ndk version you have installed ( tested with `23.2.8568313`) and `<platform>` is for android sdk platform, (tested with `android-29`)


Using your own toolchain file as described in the cmake documentation

```shell
> mkdir build-android
> cd build-android
> cmake ../ --toolchain my_toolchain.cmake
> cmake --build . -t template_release
```

Doing the equivalent on just using the command line

```shell
> mkdir build-android
> cd build-android
> cmake ../ \
  -DCMAKE_SYSTEM_NAME=Android \
  -DCMAKE_SYSTEM_VERSION=<platform> \
  -DCMAKE_ANDROID_ARCH_ABI=<arch> \
  -DCMAKE_ANDROID_NDK=/path/to/android-ndk
> cmake --build . -t template_release
```

Using the toolchain file from the Android ndk

Defaults to minimum supported version( android-16 in my case) and armv7-a.
```shell
> mkdir build-android
> cd build-android
> cmake ../ --toolchain $ANDROID_HOME/ndk/<version>/build/cmake/android.toolchain.cmake
> cmake --build . -t template_release

```

Specify Android platform and ABI

```shell
> mkdir build-android
> cd build-android
> cmake ../ --toolchain $ANDROID_HOME/ndk/<version>/build/cmake/android.toolchain.cmake \
	-DANDROID_PLATFORM:STRING=android-29 \
	-DANDROID_ABI:STRING=armeabi-v7a
> cmake --build . -t template_release
```

## Toolchains
This section attempts to list the host and target combinations that have been at tested.

Info on cross compiling triplets indicates that the naming is a little more freeform that expected, and tailored to its use case. Triplets tend to have the format `<arch>[sub][-vendor][-OS][-env]`

* [osdev.org](https://wiki.osdev.org/Target_Triplet)
* [stack overflow](https://stackoverflow.com/questions/13819857/does-a-list-of-all-known-target-triplets-in-use-exist)
* [LLVM](https://llvm.org/doxygen/classllvm_1_1Triple.html)
* [clang target triple](https://clang.llvm.org/docs/CrossCompilation.html#target-triple)
* [vcpkg](https://learn.microsoft.com/en-us/vcpkg/concepts/triplets)
* [wasm32-unknown-emscripten](https://blog.therocode.net/2020/10/a-guide-to-rust-sdl2-emscripten)

### Linux Host
Untested

### MacOS Host
tested on a mac mini, but I have to check the target triplet.

### Windows Host

#### [Microsoft Visual Studio 17 2022](https://visualstudio.microsoft.com/vs/)
	x86_64-w64

#### [LLVM](https://llvm.org/)
	x86_64-pc-windows-msvc

#### [AndroidSDK](https://developer.android.com/studio/#command-tools)
	armv7-none-linux-androideabi16

#### [Emscripten](https://emscripten.org/)
	wasm32-unknown-emscripten

#### [MinGW-w64](https://www.mingw-w64.org/) based toolchains

##### [MSYS2](https://www.msys2.org/)
Necessary reading about MSYS2 [environments](https://www.msys2.org/docs/environments/)

	ucrt64/gcc    - x86_64-w64-mingw32
	clang64/clang - x86_64-w64-mingw32

##### [LLVM-MinGW](https://github.com/mstorsjo/llvm-mingw/releases)
##### [MinGW-W64-builds](https://github.com/niXman/mingw-builds-binaries/releases)
	gcc - x86_64-w64-mingw32-ucrt
##### [Jetbrains-CLion](https://www.jetbrains.com/clion/)
	x86_64-w64-mingw32-msvcrt
