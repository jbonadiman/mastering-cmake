# How to Configure a Header File
The second approach for passing definitions to the source code is to configure a header file. The header file will include all of the `#define` macros needed to build the project. To configure a file with CMake, the [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file) command is used. This command requires an input file that is parsed by CMake to produce an output file with all variables expanded or replaced. There are three ways to specify a variable in an input file for [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file).
```sh
#cmakedefine VARIABLE
```

If VARIABLE is true, then the result will be:
```sh
#define VARIABLE
```

If VARIABLE is false, then the result will be:
```sh
/* #undef VARIABLE */
```

When writing a file to be configured, consider using `@VARIABLE@` instead of `${VARIABLE}` for variables that are expected to be expanded by CMake. Since the `${}` syntax is commonly used by other languages, users can tell the [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file) command to only expand variables using the `@var@` syntax by passing the `@ONLY` option to the command; this is useful if you are configuring a script that may contain `${var}` strings that you want to preserve. This is important because CMake will replace all occurrences of `${var}` with the empty string if `var` is not defined in CMake.

The following example configures a .h file for a project that contains preprocessor variables. The first definition indicates if the `FOOBAR` call exists in the library, and the next one contains the path to the build tree.

```sh
# ---- CMakeLists.txt file-----

# Configure a file from the source tree
# called projectConfigure.h.in and put
# the resulting configured file in the build
# tree and call it projectConfigure.h
configure_file(
   ${PROJECT_SOURCE_DIR}/projectConfigure.h.in
   ${PROJECT_BINARY_DIR}/projectConfigure.h
   @ONLY
   )
```

```sh
// -----projectConfigure.h.in file------
/* define a variable to tell the code if the */
/* foobar call is available on this system */
#cmakedefine HAS_FOOBAR_CALL

/* define a variable with the path to the */
/* build directory  */
#define PROJECT_BINARY_DIR "@PROJECT_BINARY_DIR@"
```

It is important to configure files into the binary tree, not the source tree. A single source tree may be shared by multiple build trees or platforms. By configuring files into the binary tree the differences between builds or platforms will be kept isolated in the build tree and will not corrupt other builds. This means that you will need to include the directory of the build tree where you configured the header file into the project’s list of include directories using the [`target_include_directories`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories) command.
