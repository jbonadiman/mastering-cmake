# Usage Requirements
CMake will also propagate “usage requirements” from linked library targets. Usage requirements affect compilation of sources in the \<target\>. They are specified by properties defined on linked targets.

For example, to specify include directories that are required when linking to a library you would can do the following:
```cmake
add_library(foo foo.cxx)
target_include_directories(foo PUBLIC
                           "${CMAKE_CURRENT_BINARY_DIR}"
                           "${CMAKE_CURRENT_SOURCE_DIR}"
                           )
```

Now anything that links to the target foo will automatically have foo’s binary and source as include directories. The order of the include directories brought in through “usage requirements” will match the order of the targets in the [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) call.

For each library or executable CMake creates, it tracks of all the libraries on which that target depends using the [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) command. For example:
```cmake
add_library(foo foo.cxx)
target_link_libraries(foo bar)

add_executable(foobar foobar.cxx)
target_link_libraries(foobar foo)
```

will link the libraries “foo” and “bar” into the executable “foobar” even though only “foo” was explicitly specified for it.
