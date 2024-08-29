# Hello World for CMake
To begin, let us consider the simplest possible CMakeLists file. To compile an executable from one source file, the CMakeLists file would contain three lines:
```cmake
cmake_minimum_required(VERSION 3.20)
project(Hello)
add_executable(Hello Hello.c)
```
The first line of the top level CMakeLists file should always be [`cmake_minimum_required`](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html#command:cmake_minimum_required). This allows projects to require a given version of CMake and, in addition, allows CMake to be backwards compatible.

The next line of any top level CMakeLists file should be the [`project`](https://cmake.org/cmake/help/latest/command/project.html#command:project) command. This command sets the name of the project and may specify other options such as language or version.

For each directory in a project where the CMakeLists.txt file invokes the [`project`](https://cmake.org/cmake/help/latest/command/project.html#command:project) command, CMake generates a top-level Makefile or IDE project file. The project will contain all targets that are in the CMakeLists.txt file and any subdirectories, as specified by the [`add_subdirectory`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory) command. If the [`EXCLUDE_FROM_ALL`](https://cmake.org/cmake/help/latest/prop_dir/EXCLUDE_FROM_ALL.html#prop_dir:EXCLUDE_FROM_ALL) option is used in the [`add_subdirectory`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory) command, the generated project will not appear in the top-level Makefile or IDE project file; this is useful for generating sub-projects that do not make sense as part of the main build process. Consider that a project with a number of examples could use this feature to generate the build files for each example with one run of CMake, but not have the examples built as part of the normal build process.

Finally, use the [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) command to add an executable to the project using the given source file.

In this example, there are two files in the source directory: `CMakeLists.txt` and `Hello.c`.

The next sections will describe how to configure and build the project using the CMake GUI and command line interfaces.
