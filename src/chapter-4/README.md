# CMake Cache
The CMake cache may be thought of as a configuration file. The first time CMake is run on a project, it produces a `CMakeCache.txt` file in the top directory of the build tree. CMake uses this file to store a set of global cache variables, whose values persist across multiple runs within a project build tree.

There are a few purposes of this cache. The first is to store the user’s selections and choices, so that if they should run CMake again they will not need to reenter that information. For example, the option command creates a Boolean variable and stores it in the cache.
```sh
option(USE_JPEG "Do you want to use the jpeg library")
```

The above line would create a variable called `USE_JPEG` and put it into the cache. That way the user can set that variable from the user interface and its value will remain in case the user should run CMake again in the future. To create a variable in the cache, use commands like [`option`](https://cmake.org/cmake/help/latest/command/option.html#command:option), [`find_file`](https://cmake.org/cmake/help/latest/command/find_file.html#command:find_file), or the standard [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command with the `CACHE` option.
```sh
set(USE_JPEG ON CACHE BOOL "include jpeg support?")
```

When you use the `CACHE` option, you may also provide the type of the variable and a documentation string. The type of the variable is used by the [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) to control how that variable is set and displayed, but the value is always stored in the cache file as a string.

Another purpose of the cache is to allow CMake itself to persistently store values between CMake runs. These entries may not be visible or adjustable by the user. Typically, these values are system-dependent variables that require CMake to compile and run a program to determine their value. Once these values have been determined, they are stored in the cache to avoid having to recompute them every time CMake is run. CMake generally tries to limit these variables to properties that should never change (such as the byte order of the machine you are on). If you significantly change your computer, either by changing the operating system or switching to a different compiler, you will need to delete the cache file (and probably all of your binary tree’s object files, libraries, and executables).

Some projects are very complex and setting one value in the cache may cause new options to appear the next time the cache is built. For example, VTK supports the use of MPI for performing distributed computing. This requires the build process to determine where the MPI libraries and header files are and to let the user adjust their values. But MPI is only available if another option `VTK_USE_PARALLEL` is first turned on in VTK. So, to avoid confusion for people who don’t know what MPI is, those options are hidden until `VTK_USE_PARALLEL` is turned on. So CMake shows the `VTK_USE_PARALLEL` option in the cache area, if the user turns that on and re-configures with CMake, new options will appear for MPI that they can then set. The rule is to keep building the cache until it doesn’t change. For most projects this will be just once. For some complicated ones it may be twice or more.

You might be tempted to edit the cache file directly, or to “initialize” a project by giving it a pre-populated `CMakeCache.txt` file. This may not work and could cause additional problems in the future. First, the syntax of the CMake cache is subject to change. Second, cache files contain full paths which make them unsuitable for moving between binary trees.

Once a variable is in the cache, its “cache” value cannot normally be modified from a CMakeLists file. The reasoning behind this is that once CMake has put the variable into the cache with its initial value, the user may then modify that value from the GUI. If the next invocation of CMake overwrote their change back to the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) value, the user would never be able to make a change that CMake wouldn’t overwrite. A `set(FOO ON CACHE BOOL "doc")` command will typically only do something when the cache doesn’t have the variable in it. Once the variable is in the cache, that command will have no effect.

In the rare event that you really want to change a cached variable’s value, use the `FORCE` option in combination with the `CACHE` option to the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command. The `FORCE` option will cause the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command to override and change the cache value of a variable.

A few final points should be made concerning variables and their interaction with the cache. If a variable is in the cache, it can still be overridden in a CMakeLists file using the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command without the `CACHE` option. Cache values are checked when a referenced variable is not defined in the current scope. The [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command will define a variable for the current scope without changing the value in the cache.
```sh
# assume that FOO is set to ON in the cache

set(FOO OFF)
# sets foo to OFF for processing this CMakeLists file
# and subdirectories; the value in the cache stays ON
```

Variables that are in the cache also have a property indicating if they are advanced or not. By default, when [`ccmake`](https://cmake.org/cmake/help/latest/manual/ccmake.1.html#manual:ccmake(1)) or the [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) are run, the advanced cache entries are not displayed. This is so the user can focus on the cache entries that they should consider changing. The advanced cache entries are other options that the user can modify, but typically will not. It is not unusual for a large software project to have fifty or more options, and the advanced property lets a software project divide them into key options for most users and advanced options for advanced users. Depending on the project, there may not be any non-advanced cache entries. To make a cache entry advanced, the [`mark_as_advanced`](https://cmake.org/cmake/help/latest/command/mark_as_advanced.html#command:mark_as_advanced) command is used with the name of the variable (a.k.a. cache entry).

In some cases, you might want to restrict a cache entry to a limited set of predefined options. You can do this by setting the [`STRINGS`](https://cmake.org/cmake/help/latest/prop_cache/STRINGS.html#prop_cache:STRINGS) property on the cache entry. The following CMakeLists code illustrates this by creating a property named `CRYPTOBACKEND` as usual, and then setting the `STRINGS` property on it to a set of three options.
```sh
set(CRYPTOBACKEND "OpenSSL" CACHE STRING
    "Select a cryptography backend")
set_property(CACHE CRYPTOBACKEND PROPERTY STRINGS
             "OpenSSL" "LibTomCrypt" "LibDES")
```

When [`cmake-gui`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) is run and the user selects the `CRYPTOBACKEND` cache entry, they will be presented with a pulldown to select which option they want.
