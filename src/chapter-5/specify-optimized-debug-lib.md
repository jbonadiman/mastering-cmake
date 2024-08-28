# Specifying Optimized or Debug Libraries with a Target
On Windows platforms, users are often required to link debug libraries with debug libraries, and optimized libraries with optimized libraries. CMake helps satisfy this requirement with the [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) command, which accepts an optional flag labeled as `debug` or `optimized`. If a library is preceded with either `debug` or `optimized`, then that library will only be linked in with the appropriate configuration type. For example
```sh
add_executable(foo foo.c)
target_link_libraries(foo debug libdebug optimized libopt)
```

In this case, foo will be linked against libdebug if a debug build was selected, or against libopt if an optimized build was selected.
