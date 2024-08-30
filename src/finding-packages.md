# Finding Packages
Many software projects provide tools and libraries that are meant as building blocks for other projects and applications. CMake projects that depend on outside packages locate their dependencies using the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command. A typical invocation is of the form:
```cmake
find_package(<Package> [version])
```

where `<Package>` is the name of the package to be found, and `[version]` is an optional version request (of the form `major[.minor.[patch]]`). The command’s notion of a package is distinct from that of CPack, which is meant for creating source and binary distributions and installers.

The command operates in two modes: `Module` mode and `Config` mode. In `Module` mode, the command searches for a [find module](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html#manual:cmake-developer(7)): a file named `Find<Package>.cmake`. It looks first in the [`CMAKE_MODULE_PATH`](https://cmake.org/cmake/help/latest/variable/CMAKE_MODULE_PATH.html#variable:CMAKE_MODULE_PATH) and then in the CMake installation. If a find module is found, it is loaded to search for individual components of the package. Find modules contain package-specific knowledge of the libraries and other files they expect to find, and internally use commands like [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) to locate them. CMake provides find modules for many common packages; see the [`cmake-modules(7)`](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#manual:cmake-modules(7)) manual.

The `Config` mode of [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) provides a powerful alternative through cooperation with the package to be found. It enters this mode after failing to locate a find module or when explicitly requested by the caller. In Config mode the command searches for a `package configuration file`: a file named `<Package>Config.cmake` or` <package>-config.cmake` which is provided by the package to be found. Given the name of a package, the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command knows how to search deep inside installation prefixes for locations like:
```cmake
<prefix>/lib/<package>/<package>-config.cmake
```

(see documentation of the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command for a complete list of locations). CMake creates a cache entry called `<Package>_DIR` to store the location found or allow the user to set it. Since a package configuration file comes with an installation of its package, it knows exactly where to find everything provided by the installation. Once the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command locates the file it provides the locations of package components without any additional searching.

The `[version]` option asks [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) to locate a particular version of the package. In `Module` mode, the command passes the request on to the find module. In `Config` mode the command looks next to each candidate package configuration file for a `package version file`: a file named `<Package>ConfigVersion.cmake` or `<package>-config-<version>.cmake`. The version file is loaded to test whether the package version is an acceptable match for the version requested (see documentation of [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) for the version file API specification). If the version file claims compatibility, the configuration file is accepted, or is otherwise ignored. This approach allows each project to define its own rules for version compatibility.

## Built-in Find Modules
CMake has many predefined modules that can be found in the Modules subdirectory of CMake. The modules can find many common software packages. See the [`cmake-modules(7)`](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#manual:cmake-modules(7)) manual for a detailed list.

Each `Find<XX>.cmake` module defines a set of variables that will allow a project to use the software package once it is found. Those variables all start with the name of the software being found \<XX\>. With CMake we have tried to establish a convention for naming these variables, but you should read the comments at the top of the module for a more definitive answer. The following variables are used by convention when needed:

- **\<XX\>_INCLUDE_DIRS**

    Where to find the package’s header files, typically \<XX\>.h, etc.

- **\<XX\>_LIBRARIES**

    The libraries to link against to use \<XX\>. These include full paths.

- **\<XX\>_DEFINITIONS**

    Preprocessor definitions to use when compiling code that uses \<XX\>.

- **\<XX\>_EXECUTABLE**

    Where to find the \<XX\> tool that is part of the package.

- **\<XX\>_\<YY\>_EXECUTABLE**

    Where to find the \<YY\> tool that comes with \<XX\>.

- **\<XX\>_ROOT_DIR**

    Where to find the base directory of the installation of \<XX\>. This is useful for large packages where you want to reference many files relative to a common base (or root) directory.

- **\<XX\>\_VERSION\_\<YY\>**

    Version \<YY\> of the package was found if true. Authors of find modules should make sure at most one of these is ever true. For example TCL_VERSION_84.

- **\<XX\>_\<YY\>_FOUND**

    If false, then the optional \<YY\> part of \<XX\> package is unavailable.

- **\<XX\>_FOUND**

    Set to false or undefined if we haven’t found or don’t want to use \<XX\>.

Not all of the variables are present in each of the `FindXX.cmake files`. However, the `<XX>_FOUND` should exist under most circumstances. If `<XX>` is a library, then `<XX>_LIBRARIES` should also be defined, and `<XX>_INCLUDE_DIR` should usually be defined.

Modules can be included in a project either with the [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) command or the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command.
```cmake
find_package(OpenGL)
```

is equivalent to:
```cmake
include(${CMAKE_ROOT}/Modules/FindOpenGL.cmake)
```

and
```cmake
include(FindOpenGL)
```

If the project converts over to CMake for its build system, the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) will still work if the package provides a `<XX>Config.cmake` file. How to create a CMake package is described later in this chapter.

## Creating CMake Package Configuration Files
Projects must provide package configuration files so that outside applications can find them. Consider a simple project “Gromit” providing an executable to generate source code and a library against which the generated code must link. The `CMakeLists.txt` file might start with:
```cmake
cmake_minimum_required(VERSION 3.20)
project(Gromit C)
set(version 1.0)

# Create library and executable.
add_library(gromit STATIC gromit.c gromit.h)
add_executable(gromit-gen gromit-gen.c)
```

In order to install Gromit and export its targets for use by outside projects, add the code:
```cmake
# Install and export the targets.
install(FILES gromit.h DESTINATION include/gromit-${version})
install(TARGETS gromit gromit-gen
        DESTINATION lib/gromit-${version}
        EXPORT gromit-targets)
install(EXPORT gromit-targets
        DESTINATION lib/gromit-${version})
```
Finally, Gromit must provide a package configuration file in its installation tree so that outside projects can locate it with [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package):
```cmake
# Create and install package configuration and version files.
configure_file(
   ${Gromit_SOURCE_DIR}/pkg/gromit-config.cmake.in
   ${Gromit_BINARY_DIR}/pkg/gromit-config.cmake @ONLY)

configure_file(
   ${Gromit_SOURCE_DIR}/gromit-config-version.cmake.in
   ${Gromit_BINARY_DIR}/gromit-config-version.cmake @ONLY)

install(FILES ${Gromit_BINARY_DIR}/pkg/gromit-config.cmake
         ${Gromit_BINARY_DIR}/gromit-config-version.cmake
         DESTINATION lib/gromit-${version})
```

This code configures and installs the package configuration file and a corresponding package version file. The package configuration input file `gromit-config.cmake.in` has the code:
```cmake
# Compute installation prefix relative to this file.
get_filename_component(_dir "${CMAKE_CURRENT_LIST_FILE}" PATH)
get_filename_component(_prefix "${_dir}/../.." ABSOLUTE)

# Import the targets.
include("${_prefix}/lib/gromit-@version@/gromit-targets.cmake")

# Report other information.
set(gromit_INCLUDE_DIRS "${_prefix}/include/gromit-@version@")
```

After installation, the configured package configuration file `gromit-config.cmake` knows the locations of other installed files relative to itself. The corresponding package version file is configured from its input file `gromit-config-version.cmake.in`, which contains code such as:
```cmake
set(PACKAGE_VERSION "@version@")
if(NOT "${PACKAGE_FIND_VERSION}" VERSION_GREATER "@version@")
  set(PACKAGE_VERSION_COMPATIBLE 1) # compatible with older
  if("${PACKAGE_FIND_VERSION}" VERSION_EQUAL "@version@")
    set(PACKAGE_VERSION_EXACT 1) # exact match for this version
  endif()
endif()
```

An application that uses the Gromit package might create a CMake file that looks like this:
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject C)

find_package(gromit 1.0 REQUIRED)
include_directories(${gromit_INCLUDE_DIRS})
# run imported executable
add_custom_command(OUTPUT generated.c
                   COMMAND gromit-gen generated.c)
add_executable(myexe generated.c)
target_link_libraries(myexe gromit) # link to imported library
```

The call to [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) locates an installation of Gromit or terminates with an error message if none can be found (due to `REQUIRED`). After the command succeeds, the Gromit package configuration file `gromit-config.cmake` has been loaded, so Gromit targets have been imported and variables like `gromit_INCLUDE_DIRS` have been defined.

The above example creates a package configuration file and places it in the `install` tree. One may also create a package configuration file in the `build` tree to allow applications to use the project without installation. In order to do this, one extends Gromit’s CMake file with the code:
```cmake
# Make project usable from build tree.
export(TARGETS gromit gromit-gen FILE gromit-targets.cmake)
configure_file(${Gromit_SOURCE_DIR}/gromit-config.cmake.in
               ${Gromit_BINARY_DIR}/gromit-config.cmake @ONLY)
```

This [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file) call uses a different input file, `gromit-config.cmake.in`, containing:
```cmake
# Import the targets.
include("@Gromit_BINARY_DIR@/gromit-targets.cmake")

# Report other information.
set(gromit_INCLUDE_DIRS "@Gromit_SOURCE_DIR@")
```

The package configuration file `gromit-config.cmake` placed in the build tree provides the same information to an outside project as that in the install tree, but refers to files in the source and build trees. It shares an identical package version file `gromit-config-version.cmake` which is placed in the install tree.

## CMake Package Registry
CMake provides two central locations to register packages that have been built or installed anywhere on a system: a *User Package Registry* and a *System Package Registry*. The [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command searches the two package registries as two of the search steps specified in its documentation. The registries are especially useful for helping projects find packages in non-standard install locations or directly in the package build trees. A project may populate either the user or system registry (using its own means) to refer to its location. In either case, the package should store a package configuration file at the registered location and optionally a package version file earlier in this chapter.

The *User Package Registry* is stored in a platform-specific, per-user location. On Windows it is stored in the Windows registry under a key in `HKEY_CURRENT_USER`. A `<package>` may appear under registry key
```cmake
HKEY_CURRENT_USER\Software\Kitware\CMake\Packages\<package>
```

as a `REG_SZ` value with arbitrary name that specifies the directory containing the package configuration file. On UNIX platforms, the user package registry is stored in the user home directory under `~/.cmake/packages`. A `<package>` may appear under the directory
```cmake
~/.cmake/packages/<package>
```

as a file with arbitrary name whose content specifies the directory containing the package configuration file. The [`export(PACKAGE)`](https://cmake.org/cmake/help/latest/command/export.html#command:export) command may be used to register a project build tree in the user package registry. CMake does not currently provide an interface to add install trees to the user package registry; installers must be manually taught to register their packages if desired.

The *System Package Registry* is stored in a platform-specific, system-wide location. On Windows it is stored in the Windows registry under a key in `HKEY_LOCAL_MACHINE`. A `<package>` may appear under registry key
```cmake
HKEY_LOCAL_MACHINE\Software\Kitware\CMake\Packages\<package>
```

as a `REG_SZ` value with arbitrary name that specifies the directory containing the package configuration file. There is no system package registry on non-Windows platforms. CMake does not provide an interface to add to the system package registry; installers must be manually taught to register their packages if desired.

Package registry entries are individually owned by the project installations that they reference. A package installer is responsible for adding its own entry and the corresponding uninstaller is responsible for removing it. However, in order to keep the registries clean, the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command automatically removes stale package registry entries it encounters if it has sufficient permissions. An entry is considered stale if it refers to a directory that does not exist or does not contain a matching package configuration file. This is particularly useful for user package registry entries created by the [`export(PACKAGE)`](https://cmake.org/cmake/help/latest/command/export.html#command:export) command for build trees which have no uninstall event and are simply deleted by developers.

Package registry entries may have arbitrary name. A simple convention for naming them is to use content hashes, as they are deterministic and unlikely to collide. The [`export(PACKAGE)`](https://cmake.org/cmake/help/latest/command/export.html#command:export) command uses this approach. The name of an entry referencing a specific directory is simply the content hash of the directory path itself. For example, a project may create package registry entries such as
```cmake
> reg query HKCU\Software\Kitware\CMake\Packages\MyPackage
HKEY_CURRENT_USER\Software\Kitware\CMake\Packages\MyPackage
 45e7d55f13b87179bb12f907c8de6fc4
                          REG_SZ    c:/Users/Me/Work/lib/cmake/MyPackage
 7b4a9844f681c80ce93190d4e3185db9
                          REG_SZ    c:/Users/Me/Work/MyPackage-build
```

on Windows, or
```cmake
$ cat ~/.cmake/packages/MyPackage/7d1fb77e07ce59a81bed093bbee945bd
/home/me/work/lib/cmake/MyPackage
$ cat ~/.cmake/packages/MyPackage/f92c1db873a1937f3100706657c63e07
/home/me/work/MyPackage-build
```

on UNIX. The command [`find_package(MyPackage)`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) will search the registered locations for package configuration files. The search order among package registry entries for a single package is unspecified. Registered locations may contain package version files to tell [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) whether a specific location is suitable for the version requested.
