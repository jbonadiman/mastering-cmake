# Utility Modules
Utility modules are simply sections of CMake commands put into a file; they can then be included into other CMakeLists files using the [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) command. For example, the following commands will include the [`CheckTypeSize`](https://cmake.org/cmake/help/latest/module/CheckTypeSize.html#module:CheckTypeSize) module from CMake and then use the macro it defines.
```cmake
include(CheckTypeSize)
check_type_size(long SIZEOF_LONG)
```

These modules test the system to provide information about the target platform or compiler, such as the size of a float or support for ANSI C++ streams. Many of these modules have names prefixed with `Test` or `Check`, such as [`TestBigEndian`](https://cmake.org/cmake/help/latest/module/TestBigEndian.html#module:TestBigEndian) and [`CheckTypeSize`](https://cmake.org/cmake/help/latest/module/CheckTypeSize.html#module:CheckTypeSize). Some of them try to compile code in order to determine the correct result. In these cases, the source code is typically named the same as the module, but with a `.c` or `.cxx` extension. Utility modules also provide useful macros and functions implemented in the CMake language and intended for specific, common use cases. See documentation of each module for details.
