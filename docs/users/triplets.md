# Triplet files

Triplet is a standard term used in cross compiling as a way to completely capture the target environment (cpu, os, compiler, runtime, etc) in a single convenient name.

In Vcpkg, we use triplets to describe an imaginary "target configuration set" for every library. Within a triplet, libraries are generally built with the same configuration, but it is not a requirement. For example, you could have one triplet that builds `openssl` statically and `zlib` dynamically, one that builds them both statically, and one that builds them both dynamically (all for the same target OS and architecture). A single build will consume files from a single triplet.

We currently provide many triplets by default (run `vcpkg help triplet`). However, you can easily add your own by creating a new file in the `triplets\` directory. The new triplet will immediately be available for use in commands, such as `vcpkg install boost:x86-windows-custom`.

To change the triplet used by your project, such as to enable static linking, see our [Integration Document](integration.md#triplet-selection).

## Community triplets

Triplets contained in the `triplets\community` folder are not tested by continuous integration, but are commonly requested by the community.

Because we do not have continuous coverage, port updates may break compatibility with community triplets. Because of this, community involvement is paramount!

We will gladly accept and review contributions that aim to solve issues with these triplets.

### Usage

Community Triplets are enabled by default, when using a community triplet a message like the following one will be printed during a package install:

```no-highlight
-- Using community triplet x86-uwp. This triplet configuration is not guaranteed to succeed.
-- [COMMUNITY] Loading triplet configuration from: D:\src\viromer\vcpkg\triplets\community\x86-uwp.cmake
```

## Variables
### VCPKG_TARGET_ARCHITECTURE
Specifies the target machine architecture.

Valid options are `x86`, `x64`, `arm`, and `arm64`.

### VCPKG_CRT_LINKAGE
Specifies the desired CRT linkage (for MSVC).

Valid options are `dynamic` and `static`.

### VCPKG_LIBRARY_LINKAGE
Specifies the preferred library linkage.

Valid options are `dynamic` and `static`. Note that libraries can ignore this setting if they do not support the preferred linkage type.

### VCPKG_CMAKE_SYSTEM_NAME
Specifies the target platform.

Valid options include any CMake system name, such as:
- Empty (Windows Desktop for legacy reasons)
- `WindowsStore` (Universal Windows Platform)
- `Darwin` (Mac OSX)
- `Linux` (Linux)

### VCPKG_CMAKE_SYSTEM_VERSION
Specifies the target platform system version.

This field is optional and, if present, will be passed into the build as `CMAKE_SYSTEM_VERSION`.

See also the CMake documentation for `CMAKE_SYSTEM_VERSION`: https://cmake.org/cmake/help/latest/variable/CMAKE_SYSTEM_VERSION.html.

### VCPKG_CHAINLOAD_TOOLCHAIN_FILE
Specifies an alternate CMake Toolchain file to use.

This (if set) will override all other compiler detection logic. By default, a toolchain file is selected from `scripts/toolchains/` appropriate to the platform.

See also the CMake documentation for toolchain files: https://cmake.org/cmake/help/v3.11/manual/cmake-toolchains.7.html.

### VCPKG_CXX_FLAGS
Sets additional compiler flags to be used when not using `VCPKG_CHAINLOAD_TOOLCHAIN_FILE`.

This option also has forms for configuration-specific and C flags:
- `VCPKG_CXX_FLAGS`
- `VCPKG_CXX_FLAGS_DEBUG`
- `VCPKG_CXX_FLAGS_RELEASE`
- `VCPKG_C_FLAGS`
- `VCPKG_C_FLAGS_DEBUG`
- `VCPKG_C_FLAGS_RELEASE`

<a name="VCPKG_DEP_INFO_OVERRIDE_VARS"></a>
### VCPKG_DEP_INFO_OVERRIDE_VARS
Replaces the default computed list of triplet "Supports" terms.

This option (if set) will override the default set of terms used for qualified dependency resolution and "Supports" field evaluation.

See the [`Supports`](../maintainers/control-files.md#Supports) control file field documentation for more details.

> Implementers' Note: this list is extracted via the `vcpkg_get_dep_info` mechanism.

## Windows Variables

### VCPKG_ENV_PASSTHROUGH
Instructs vcpkg to allow additional environment variables into the build process.

On Windows, vcpkg builds packages in a special clean environment that is isolated from the current command prompt to ensure build reliability and consistency.

This triplet option can be set to a list of additional environment variables that will be added to the clean environment.

See also the `vcpkg env` command for how you can inspect the precise environment that will be used.

> Implementers' Note: this list is extracted via the `vcpkg_get_tags` mechanism.

<a name="VCPKG_VISUAL_STUDIO_PATH"></a>
### VCPKG_VISUAL_STUDIO_PATH
Specifies the Visual Studio installation to use.

To select the precise combination of Visual Studio instance and toolset version, we walk through the following algorithm:
1. Determine the setting for `VCPKG_VISUAL_STUDIO_PATH` from the triplet, or the environment variable `VCPKG_VISUAL_STUDIO_PATH`, or consider it unset
2. Determine the setting for `VCPKG_PLATFORM_TOOLSET` from the triplet or consider it unset
3. Gather a list of all pairs of Visual Studio Instances with all toolsets available in those instances
    1. This is ordered first by instance type (Stable, Prerelease, Legacy) and then by toolset version (v142, v141, v140)
4. Filter the list based on the settings for `VCPKG_VISUAL_STUDIO_PATH` and `VCPKG_PLATFORM_TOOLSET`.
5. Select the best remaining option

The path should be absolute, formatted with backslashes, and have no trailing slash:
```cmake
set(VCPKG_VISUAL_STUDIO_PATH "C:\\Program Files (x86)\\Microsoft Visual Studio\\Preview\\Community")
```

### VCPKG_PLATFORM_TOOLSET
Specifies the VS-based C/C++ compiler toolchain to use.

See [`VCPKG_VISUAL_STUDIO_PATH`](#VCPKG_VISUAL_STUDIO_PATH) for the full selection algorithm.

Valid settings:
* The Visual Studio 2019 platform toolset is `v142`.
* The Visual Studio 2017 platform toolset is `v141`.
* The Visual Studio 2015 platform toolset is `v140`.

### VCPKG_LOAD_VCVARS_ENV
If `VCPKG_CHAINLOAD_TOOLCHAIN_FILE` is used, VCPKG will not setup the Visual Studio environment. 
Setting `VCPKG_LOAD_VCVARS_ENV` to (true|1|on) changes this behavior so that the Visual Studio environment is setup following the same rules as if `VCPKG_CHAINLOAD_TOOLCHAIN_FILE` was not set.

## MacOS Variables

### VCPKG_INSTALL_NAME_DIR
Sets the install name used when building macOS dynamic libraries. Default value is `@rpath`. See the CMake documentation for [CMAKE_INSTALL_NAME_DIR](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_NAME_DIR.html) for more information.

### VCPKG_OSX_DEPLOYMENT_TARGET
Sets the minimum macOS version for compiled binaries. This also changes what versions of the macOS platform SDK that CMake will search for. See the CMake documentation for [CMAKE_OSX_DEPLOYMENT_TARGET](https://cmake.org/cmake/help/latest/variable/CMAKE_OSX_DEPLOYMENT_TARGET.html) for more information.

### VCPKG_OSX_SYSROOT
Set the name or path of the macOS platform SDK that will be used by CMake. See the CMake documentation for [CMAKE_OSX_SYSROOT](https://cmake.org/cmake/help/latest/variable/CMAKE_OSX_SYSROOT.html) for more information.


### VCPKG_OSX_ARCHITECTURES
Set the macOS / iOS target architecture which will be used by CMake. See the CMake documentation for [CMAKE_OSX_ARCHITECTURES](https://cmake.org/cmake/help/latest/variable/CMAKE_OSX_ARCHITECTURES.html) for more information.

## Per-port customization
The CMake Macro `PORT` will be set when interpreting the triplet file and can be used to change settings (such as `VCPKG_LIBRARY_LINKAGE`) on a per-port basis.

Example:
```cmake
set(VCPKG_LIBRARY_LINKAGE static)
if(PORT MATCHES "qt5-")
    set(VCPKG_LIBRARY_LINKAGE dynamic)
endif()
```
This will build all the `qt5-*` libraries as DLLs, but every other library as a static library.

For an example in a real project, see https://github.com/Intelight/vcpkg/blob/master/triplets/x86-windows-mixed.cmake.

## Additional Remarks
The default triplet when running any vcpkg command is `%VCPKG_DEFAULT_TRIPLET%` or a platform-specific choice if that environment variable is undefined.

- Windows: `x86-windows`
- Linux: `x64-linux`
- OSX: `x64-osx`

We recommend using a systematic naming scheme when creating new triplets. The Android toolchain naming scheme is a good source of inspiration: https://developer.android.com/ndk/guides/standalone_toolchain.html.
