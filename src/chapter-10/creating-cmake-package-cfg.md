# Creating CMake Package Configuration Files
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
