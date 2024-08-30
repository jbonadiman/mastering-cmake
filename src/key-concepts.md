# Key Concepts
This chapter provides an introduction to CMake’s key concepts. As you start working with CMake, you will run into a variety of concepts such as targets, generators, and commands. Understanding these concepts will provide you with the working knowledge you need to create effective CMakeLists files. Many CMake objects such as targets, directories and source files have properties associated with them. A property is a key-value pair attached to a specific object. The most generic way to access properties is through the [`set_property`](https://cmake.org/cmake/help/latest/command/set_property.html#command:set_property) and [`get_property`](https://cmake.org/cmake/help/latest/command/get_property.html#command:get_property) commands. These commands allow you to set or get a property from any object in CMake that has properties. See the [`cmake-properties`](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html#manual:cmake-properties(7)) manual for a list of supported properties. From the command line a full list of properties supported in CMake can be obtained by running [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1)) with the `--help-property-list` option.

## Targets
Probably the most important item is targets. Targets represent executables, libraries, and utilities built by CMake. Every [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library), [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable), and [`add_custom_target`](https://cmake.org/cmake/help/latest/command/add_custom_target.html#command:add_custom_target) command creates a target. For example, the following command will create a target named “foo” that is a static library, with `foo1.c` and `foo2.c` as source files.
```cmake
add_library(foo STATIC foo1.c foo2.c)
```

The name “foo” is now available for use as a library name everywhere else in the project, and CMake will know how to expand the name into the library when needed. Libraries can be declared as a particular type such as `STATIC`, `SHARED`, `MODULE`, or left undeclared. `STATIC` indicates that the library must be built as a static library. Likewise, `SHARED` indicates it must be built as a shared library. `MODULE` indicates that the library must be created so that it can be dynamically-loaded into an executable. Module libraries are implemented as shared libraries on many platforms, but not all. Therefore, CMake does not allow other targets to link to modules. If none of these options are specified, it indicates that the library could be built as either shared or static. In that case, CMake uses the setting of the variable [`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS) to determine if the library should be `SHARED` or `STATIC`. If it is not set, then CMake defaults to building static libraries.

Likewise, executables have some options. By default, an executable will be a traditional console application that has a main entry point. One may specify a `WIN32` option to request a WinMain entry point on Windows systems, while retaining main on non-Windows systems.

In addition to storing their type, targets also keep track of general properties. These properties can be set and retrieved using the [`set_target_properties`](https://cmake.org/cmake/help/latest/command/set_target_properties.html#command:set_target_properties) and [`get_target_property`](https://cmake.org/cmake/help/latest/command/get_target_property.html#command:get_target_property) commands, or the more general [`set_property`](https://cmake.org/cmake/help/latest/command/set_property.html#command:set_property) and [`get_property`](https://cmake.org/cmake/help/latest/command/get_property.html#command:get_property) commands. One useful property is [`LINK_FLAGS`](https://cmake.org/cmake/help/latest/prop_tgt/LINK_FLAGS.html#prop_tgt:LINK_FLAGS), which is used to specify additional link flags for a specific target. Targets store a list of libraries that they link against, which are set using the [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) command. Names passed into this command can be libraries, full paths to libraries, or the name of a library from an [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) command. Targets also store the link directories to use when linking, and custom commands to execute after building.

## Usage Requirements
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

## Specifying Optimized or Debug Libraries with a Target
On Windows platforms, users are often required to link debug libraries with debug libraries, and optimized libraries with optimized libraries. CMake helps satisfy this requirement with the [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html#command:target_link_libraries) command, which accepts an optional flag labeled as `debug` or `optimized`. If a library is preceded with either `debug` or `optimized`, then that library will only be linked in with the appropriate configuration type. For example
```cmake
add_executable(foo foo.c)
target_link_libraries(foo debug libdebug optimized libopt)
```

In this case, foo will be linked against libdebug if a debug build was selected, or against libopt if an optimized build was selected.

## Object Libraries
Large projects often organize their source files into groups, perhaps in separate subdirectories, that each need different include directories and preprocessor definitions. For this use case CMake has developed the concept of Object Libraries.

An Object Library is a collection of source files compiled into an object file which is not linked into a library file or made into an archive. Instead other targets created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) or [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) may reference the objects using an expression of the form `$<TARGET_OBJECTS:name>` as a source, where “name” is the target created by the [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) call. For example:
```cmake
add_library(A OBJECT a.cpp)
add_library(B OBJECT b.cpp)
add_library(Combined $<TARGET_OBJECTS:A> $<TARGET_OBJECTS:B>)
```

will include A and B object files in a library called Combined. Object libraries may contain only sources (and headers) that compile to object files.

## Source Files
The source file structure is in many ways similar to a target. It stores the filename, extension, and a number of general properties related to a source file. Like targets, you can set and get properties using [`set_source_files_properties`](https://cmake.org/cmake/help/latest/command/set_source_files_properties.html#command:set_source_files_properties) and [`get_source_file_property`](https://cmake.org/cmake/help/latest/command/get_source_file_property.html#command:get_source_file_property), or the more generic versions.

## Directories, Tests, and Properties
In addition to targets and source files, you may find yourself occasionally working with other objects such as directories and tests. Normally such interactions take the shape of setting or getting properties (e.g. [`set_directory_properties`](https://cmake.org/cmake/help/latest/command/set_directory_properties.html#command:set_directory_properties) or [`set_tests_properties`](https://cmake.org/cmake/help/latest/command/set_tests_properties.html#command:set_tests_properties)) from these objects.
