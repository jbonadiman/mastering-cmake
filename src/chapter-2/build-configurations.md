# Build Configurations
Build configurations allow a project to be built in different ways for debug, optimized, or any other special set of flags. CMake supports, by default, Debug, Release, MinSizeRel, and RelWithDebInfo configurations. Debug has the basic debug flags turned on. Release has the basic optimizations turned on. MinSizeRel has flags that produce the smallest object code, but not necessarily the fastest code. RelWithDebInfo builds an optimized build with debug information as well.

CMake handles the configurations in slightly different ways depending on the generator being used. The conventions of the native build system are followed when possible. This means that configurations impact the build in different ways when using Makefiles versus using Visual Studio project files.

The Visual Studio IDE supports the notion of Build Configurations. A default project in Visual Studio usually has Debug and Release configurations. From the IDE you can select build Debug, and the files will be built with Debug flags. The IDE puts all of the binary files into directories with the name of the active configuration. This brings about an extra complexity for projects that build programs that need to be run as part of the build process from custom commands. See the [`CMAKE_CFG_INTDIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_CFG_INTDIR.html#variable:CMAKE_CFG_INTDIR) variable and the custom commands section for more information about how to handle this issue. The variable [`CMAKE_CONFIGURATION_TYPES`](https://cmake.org/cmake/help/latest/variable/CMAKE_CONFIGURATION_TYPES.html#variable:CMAKE_CONFIGURATION_TYPES) is used to tell CMake which configurations to put in the workspace.

With Makefile-based generators, only one configuration can be active at the time CMake is run, and it is specified with the [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html#variable:CMAKE_BUILD_TYPE) variable. If the variable is empty then no flags are added to the build. If the variable is set to the name of a configuration, then the appropriate variables and rules (such as `CMAKE_CXX_FLAGS_<ConfigName>`) are added to the compile lines. Makefiles do not use special configuration subdirectories for object files. To build both debug and release trees, the user is expected to create multiple build directories using the out-of-source build feature of CMake, and set the [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html#variable:CMAKE_BUILD_TYPE) to the desired selection for each build. For example:
```cmake
# With source code in the directory MyProject
# to build MyProject-debug create that directory, cd into it and
ccmake ../MyProject -DCMAKE_BUILD_TYPE=Debug
# the same idea is used for the release tree MyProject-release
ccmake ../MyProject -DCMAKE_BUILD_TYPE=Release
```
