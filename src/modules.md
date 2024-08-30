# Modules

## Using Modules
Code reuse is a valuable technique in software development and CMake has been designed to support it. Allowing CMakeLists files to make use of reusable modules enables the entire community to share reusable sections of code. For CMake, these sections are called [`cmake-modules`](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#manual:cmake-modules(7)) and can be found in the Modules subdirectory of your installation.

A module’s location can be specified using the full path to the module file, or by letting CMake find the module by itself. CMake will look for modules in the directories specified by [`CMAKE_MODULE_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_MODULE_PATH.html#variable:CMAKE_MODULE_PATH); if it cannot find it there, it will look in the Modules subdirectory. This way projects can override modules that CMake provides and customize them for their needs. Modules can be broken into a few main categories.

### Find Modules
These modules support the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command to determine the location of software elements, such as header files or libraries, that belong to a given package. Do not include them directly. Use the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command. Each module comes with documentation describing the package it finds and the variables in which it provides results.

### Utility Modules
Utility modules are simply sections of CMake commands put into a file; they can then be included into other CMakeLists files using the [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) command. For example, the following commands will include the [`CheckTypeSize`](https://cmake.org/cmake/help/latest/module/CheckTypeSize.html#module:CheckTypeSize) module from CMake and then use the macro it defines.
```cmake
include(CheckTypeSize)
check_type_size(long SIZEOF_LONG)
```

These modules test the system to provide information about the target platform or compiler, such as the size of a float or support for ANSI C++ streams. Many of these modules have names prefixed with `Test` or `Check`, such as [`TestBigEndian`](https://cmake.org/cmake/help/latest/module/TestBigEndian.html#module:TestBigEndian) and [`CheckTypeSize`](https://cmake.org/cmake/help/latest/module/CheckTypeSize.html#module:CheckTypeSize). Some of them try to compile code in order to determine the correct result. In these cases, the source code is typically named the same as the module, but with a `.c` or `.cxx` extension. Utility modules also provide useful macros and functions implemented in the CMake language and intended for specific, common use cases. See documentation of each module for details.
