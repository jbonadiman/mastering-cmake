# Using Header Files and Libraries
Many C and C++ programs depend on external libraries; however, when it comes to the practical aspects of compiling and linking a project, taking advantage of existing libraries can be difficult for both developers and users. Problems typically show up as soon as the software is built on a system other than the one on which it was developed. Assumptions regarding where libraries and header files are located become obvious when they are not installed in the same place on the new computer and the build system is unable to find them. CMake has many features to aid developers in the integration of external software libraries into a project.

The CMake commands that are most relevant to this type of integration are the [`find_file`](https://cmake.org/cmake/help/latest/command/find_file.html#command:find_file), [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library), [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path), [`find_program`](https://cmake.org/cmake/help/latest/command/find_program.html#command:find_program), and [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) commands. For most C and C++ libraries, a combination of [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) and [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) will be enough to compile and link with an installed library. The command [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) can be used to locate, or allow a user to locate a library, and [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) can be used to find the path to a representative include file from the project. For example, if you wanted to link to the tiff library, you could use the following commands in your CMakeLists.txt file
```sh
# find libtiff, looking in some standard places
find_library(TIFF_LIBRARY
             NAMES tiff tiff2
             PATHS /usr/local/lib /usr/lib
            )

# find tiff.h looking in some standard places
find_path(TIFF_INCLUDES tiff.h
           /usr/local/include
           /usr/include
          )

add_executable(mytiff mytiff.c )

target_link_libraries(mytiff ${TIFF_LIBRARY})

target_include_directories(mytiff ${TIFF_INCLUDES})
```

The first command used is [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library), which in this case, will look for a library with the name tiff or tiff2. The [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) command only requires the base name of the library without any platform-specific prefixes or suffixes, such as .lib and .dll. The appropriate prefixes and suffixes for the system running CMake will be added to the library name automatically when CMake attempts to find it. All the `FIND_*` commands will look in the `PATH` environment variable. In addition, the commands allow the specification of additional search paths as arguments to be listed after the `PATHS` marker argument. In addition to supporting standard paths, Windows registry entries and environment variables can be used to construct search paths. The syntax for registry entries is the following:
```sh
[HKEY_CURRENT_USER\\Software\\Kitware\\Path;Build1]
```

Since software can be installed in many different places, it is impossible for CMake to find the library every time, but most standard installations should be covered. The `find_*` commands automatically create a cache variable so that users can override or specify the location from the CMake GUI. This way, if CMake is unable to locate the files it is looking for, users will still have an opportunity to specify them. If CMake does not find a file, the value is set to `VAR-NOTFOUND`; this value tells CMake that it should continue looking each time CMake’s configure step is run. Note that in if statements, values of `VAR-NOTFOUND` will evaluate as false.

The next command used is [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path), a general purpose command that, in this example, is used to locate a header file from the library. Header files and libraries are often installed in different locations, and both locations are required to compile and link programs that use them. The use of [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) is similar to [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library), although it only supports one name, a list of search paths.

The remainder of the CMakeLists file may use the variables created by the `find_*` commands. The variables can be used without checking for valid values, as CMake will print an error message notifying the user if any of the required variables have not been set. The user can then set the cache values and reconfigure until the message goes away. Optionally, a CMakeLists file could use the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command to use alternative libraries or options to build the project without the library if it cannot be found.

From the above example you can see how using the `find_*` commands can help your software compile on a variety of systems. It is worth noting that the `find_*` commands search for a match starting with the first argument and first path, so when listing paths and library names, list your preferred paths and names first. If there are multiple versions of a library and you would prefer tiff over tiff2, make sure they are listed in that order.
