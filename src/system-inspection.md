# System Inspection
This chapter will describe how to use CMake to inspect the environment of the system where the software is being built. This is a critical factor in creating cross-platform applications or libraries. It covers how to find and use system and user installed header files and libraries. It also covers some of the more advanced features of CMake, including the [`try_compile`](https://cmake.org/cmake/help/latest/command/try_compile.html#command:try_compile) and [`try_run`](https://cmake.org/cmake/help/latest/command/try_run.html#command:try_run) commands. These commands are extremely powerful tools for determining the capabilities of the system and compiler that is hosting your software.

## Using Header Files and Libraries
Many C and C++ programs depend on external libraries; however, when it comes to the practical aspects of compiling and linking a project, taking advantage of existing libraries can be difficult for both developers and users. Problems typically show up as soon as the software is built on a system other than the one on which it was developed. Assumptions regarding where libraries and header files are located become obvious when they are not installed in the same place on the new computer and the build system is unable to find them. CMake has many features to aid developers in the integration of external software libraries into a project.

The CMake commands that are most relevant to this type of integration are the [`find_file`](https://cmake.org/cmake/help/latest/command/find_file.html#command:find_file), [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library), [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path), [`find_program`](https://cmake.org/cmake/help/latest/command/find_program.html#command:find_program), and [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) commands. For most C and C++ libraries, a combination of [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) and [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) will be enough to compile and link with an installed library. The command [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library) can be used to locate, or allow a user to locate a library, and [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) can be used to find the path to a representative include file from the project. For example, if you wanted to link to the tiff library, you could use the following commands in your CMakeLists.txt file
```cmake
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
```cmake
[HKEY_CURRENT_USER\\Software\\Kitware\\Path;Build1]
```

Since software can be installed in many different places, it is impossible for CMake to find the library every time, but most standard installations should be covered. The `find_*` commands automatically create a cache variable so that users can override or specify the location from the CMake GUI. This way, if CMake is unable to locate the files it is looking for, users will still have an opportunity to specify them. If CMake does not find a file, the value is set to `VAR-NOTFOUND`; this value tells CMake that it should continue looking each time CMake’s configure step is run. Note that in if statements, values of `VAR-NOTFOUND` will evaluate as false.

The next command used is [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path), a general purpose command that, in this example, is used to locate a header file from the library. Header files and libraries are often installed in different locations, and both locations are required to compile and link programs that use them. The use of [`find_path`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) is similar to [`find_library`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library), although it only supports one name, a list of search paths.

The remainder of the CMakeLists file may use the variables created by the `find_*` commands. The variables can be used without checking for valid values, as CMake will print an error message notifying the user if any of the required variables have not been set. The user can then set the cache values and reconfigure until the message goes away. Optionally, a CMakeLists file could use the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command to use alternative libraries or options to build the project without the library if it cannot be found.

From the above example you can see how using the `find_*` commands can help your software compile on a variety of systems. It is worth noting that the `find_*` commands search for a match starting with the first argument and first path, so when listing paths and library names, list your preferred paths and names first. If there are multiple versions of a library and you would prefer tiff over tiff2, make sure they are listed in that order.

## System Properties
Although it is a common practice in C and C++ code to add platform-specific code inside preprocessor `ifdef` directives, for maximum portability this should be avoided. Software should not be tuned to specific platforms with `ifdefs`, but rather to a canonical system consisting of a set of features. Coding to specific systems makes the software less portable, because systems and the features they support change with time, and even from system to system. A feature that may not have worked on a platform in the past may be a required feature for the platform in the future. The following code fragments illustrate the difference between coding to a canonical system and a specific system:
```cmake
// coding to a feature
#ifdef HAS_FOOBAR_CALL
  foobar();
#else
  myfoobar();
#endif

// coding to specific platforms
#if defined(SUN) && defined(HPUX) && !defined(GNUC)
  foobar();
#else
  myfoobar();
#endif
```
The problem with the second approach is that the code will have to be modified for each new platform on which the software is compiled. For example, a future version of SUN may no longer have the foobar call. Using the `HAS_FOOBAR_CALL` approach, the software will work as long as `HAS_FOOBAR_CALL` is defined correctly, and this is where CMake can help. CMake can be used to define `HAS_FOOBAR_CALL` correctly and automatically by making use of the [`try_compile`](https://cmake.org/cmake/help/latest/command/try_compile.html#command:try_compile) and [`try_run`](https://cmake.org/cmake/help/latest/command/try_run.html#command:try_run) commands. These commands can be used to compile and run small test programs during the CMake configure step. The test programs will be sent to the compiler that will be used to build the project, and if errors occur, the feature can be disabled. These commands require that you write a small C or C++ program to test the feature. For example, to test if the `foobar` call is provided on the system, try compiling a simple program that uses `foobar`. First write the simple test program (testNeedFoobar.c in this example) and then add the CMake calls to the CMakeLists file to try compiling that code. If the compilation works then `HAS_FOOBAR_CALL` will be set to true.
```cmake
// --- testNeedFoobar.c -----

#include <foobar.h>
main()
{
  foobar();
}
```

```cmake
# --- testNeedFoobar.cmake ---

try_compile (HAS_FOOBAR_CALL
  ${CMAKE_BINARY_DIR}
  ${PROJECT_SOURCE_DIR}/testNeedFoobar.c
  )
```

Now that `HAS_FOOBAR_CALL` is set correctly in CMake, you can use it in your source code through the [`target_compile_definitions`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions) command. Alternatively, it is possible to configure a header file. This is discussed further in the section called [How to Configure a Header File](./configure-header-file.md).

Sometimes compiling a test program is not enough. In some cases, you may actually want to compile and run a program to get its output. A good example of this is testing the byte order of a machine. The following example shows how to write a small program that CMake will compile and run to determine the byte order of a machine.
```cmake
// ---- TestByteOrder.c ------

int main () {
  /* Are we most significant byte first or last */
  union
  {
    long l;
    char c[sizeof (long)];
  } u;
  u.l = 1;
  exit (u.c[sizeof (long) - 1] == 1);
}
```

```cmake
# ----- TestByteOrder.cmake-----

try_run(RUN_RESULT_VAR
  COMPILE_RESULT_VAR
  ${CMAKE_BINARY_DIR}
  ${PROJECT_SOURCE_DIR}/Modules/TestByteOrder.c
  OUTPUT_VARIABLE OUTPUT
  )
```

The return result of the run will go into `RUN_RESULT_VAR`, the result of the compile will go into `COMPILE_RESULT_VAR`, and any output from the run will go into `OUTPUT`. You can use these variables to report debug information to the users of your project.

For small test programs the [`file`](https://cmake.org/cmake/help/latest/command/file.html#command:file) command with the `WRITE` option can be used to create the source file from the CMakeLists file. The following example tests the C compiler to verify that it can be run.
```cmake
file(WRITE
  ${CMAKE_BINARY_DIR}/CMakeTmp/testCCompiler.c
  "int main(){return 0;}"
  )

try_compile(CMAKE_C_COMPILER_WORKS
  ${CMAKE_BINARY_DIR}
  ${CMAKE_BINARY_DIR}/CMakeTmp/testCCompiler.c
  OUTPUT_VARIABLE OUTPUT
  )
```

For more advanced [`try_compile`](https://cmake.org/cmake/help/latest/command/try_compile.html#command:try_compile) and [`try_run`](https://cmake.org/cmake/help/latest/command/try_run.html#command:try_run) operations, it may be desirable to pass flags to the compiler or to CMake. Both commands support the optional arguments `CMAKE_FLAGS` and `COMPILE_DEFINITIONS`. `CMAKE_FLAGS` can be used to pass `-DVAR:TYPE=VALUE` flags to CMake. The value of `COMPILE_DEFINITIONS` is passed directly to the compiler command line.

There are several predefined try-run and try-compile modules available in CMake [`cmake-modules(7)`](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html#manual:cmake-modules(7)), some of which are listed below. These modules allow some common checks to be performed without having to create a source file for each test. Many of these modules will look at the current value of the `CMAKE_REQUIRED_FLAGS` and `CMAKE_REQUIRED_LIBRARIES` variables to add additional compile flags or link libraries to the test.

- [`CheckIncludeFile`](https://cmake.org/cmake/help/latest/module/CheckIncludeFile.html#module:CheckIncludeFile)

    Provides a macro that checks for an include file on a system by taking two arguments with first being the include file to look for and the second being the variable to store the result into. Additional CFlags can be passed in as a third argument or by setting `CMAKE_REQUIRED_FLAGS`.

- [`CheckIncludeFileCXX`](https://cmake.org/cmake/help/latest/module/CheckIncludeFileCXX.html#module:CheckIncludeFileCXX)

    Provides a macro that checks for an include file in a C++ program by taking two arguments with the first being the include file to look for and the second being the variable to store the result into. Additional CFlags can be passed in as a third argument.

- [`CheckIncludeFiles`](https://cmake.org/cmake/help/latest/module/CheckIncludeFiles.html#module:CheckIncludeFiles)

    Provides a macro that checks if the given header files can be include together. This macro uses `CMAKE_REQUIRED_FLAGS` if it is set, and is useful when a header file you are interested in checking for is dependent on including another header file first.

- [`CheckLibraryExists`](https://cmake.org/cmake/help/latest/module/CheckLibraryExists.html#module:CheckLibraryExists)

    Provides a macro that checks to see if a library exists by taking four arguments with the first being the name of the library to check for; the second being the name of a function that should be in that library; the third argument being the location of where the library should be found; and the fourth argument being a variable to store the result into. This macro uses `CMAKE_REQUIRED_FLAGS` and `CMAKE_REQUIRED_LIBRARIES` if they are set.

- [`CheckSymbolExists`](https://cmake.org/cmake/help/latest/module/CheckSymbolExists.html#module:CheckSymbolExists)

    Provides a macro that checks to see if a symbol is defined in a header file by taking three arguments with the first being the symbol to look for; the second argument being a list of header files to try including; and the third argument being where the result is stored. This macro uses `CMAKE_REQUIRED_FLAGS` and `CMAKE_REQUIRED_LIBRARIES` if they are set.

- [`CheckTypeSize`](https://cmake.org/cmake/help/latest/module/CheckTypeSize.html#module:CheckTypeSize)

    Provides a macro that determines the size in bytes of a variable type by taking two arguments with the first argument being the type to evaluate, and the second argument being where the result is stored. Both `CMAKE_REQUIRED_FLAGS` and `CMAKE_REQUIRED_LIBRARIES` are used if they are set.

- [`CheckVariableExists`](https://cmake.org/cmake/help/latest/module/CheckVariableExists.html#module:CheckVariableExists)

    Provides a macro that checks to see if a global variable exists by taking two arguments with the first being the variable to look for, and the second argument being the variable to store the result in. This macro will prototype the named variable and then try to use it. If the test program compiles then the variable exists. This will only work for C variables. This macro uses `CMAKE_REQUIRED_FLAGS` and `CMAKE_REQUIRED_LIBRARIES` if they are set.

Consider the following example which shows a variety of these modules being used to compute properties of the platform. At the beginning of the example four modules are loaded from CMake. The remainder of the example uses the macros defined in those modules to test for header files, libraries, symbols, and type sizes respectively.
```cmake
# Include all the necessary files for macros
include(CheckIncludeFiles)
include(CheckLibraryExists)
include(CheckSymbolExists)
include(CheckTypeSize)

# Check for header files
set(INCLUDES "")
check_include_files("${INCLUDES};winsock.h" HAVE_WINSOCK_H)

if(HAVE_WINSOCK_H)
  set(INCLUDES ${INCLUDES} winsock.h)
endif()

check_include_files("${INCLUDES};io.h" HAVE_IO_H)
if (HAVE_IO_H)
  set(INCLUDES ${INCLUDES} io.h)
endif()

# Check for all needed libraries
set(LIBS "")
check_library_exists("dl;${LIBS}" dlopen "" HAVE_LIBDL)
if(HAVE_LIBDL)
  set(LIBS ${LIBS} dl)
endif()

check_library_exists("ucb;${LIBS}" gethostname "" HAVE_LIBUCB)
if(HAVE_LIBUCB)
  set(LIBS ${LIBS} ucb)
endif()

# Add the libraries we found to the libraries to use when
# looking for symbols with the check_symbol_exists macro
set(CMAKE_REQUIRED_LIBRARIES ${LIBS})

# Check for some functions that are used
check_symbol_exists(socket "${INCLUDES}" HAVE_SOCKET)
check_symbol_exists(poll "${INCLUDES}" HAVE_POLL)

# Various type sizes
check_type_size(int SIZEOF_INT)
check_type_size(size_t SIZEOF_SIZE_T)
```

## How to Pass Parameters to a Compilation
Once you have determined the features of the system in which you are interested, it is time to configure the software based on what has been found. There are two common ways to pass this information to the compiler: on the compile line, or using a pre-configured header. The first way is to pass definitions on the compile line. A preprocessor definition can be passed to the compiler from a CMakeLists file with the [`target_compile_definitions`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions) command. For example, a common practice in C code is to have the ability to selectively compile in/out debug statements.
```cmake
#ifdef DEBUG_BUILD
  printf("the value of v is %d", v);
#endif
```

A CMake variable could be used to turn on or off debug builds using the [`option`](https://cmake.org/cmake/help/latest/command/option.html#command:option) command:
```cmake
option(DEBUG_BUILD
      "Build with extra debug print messages.")

if(DEBUG_BUILD)
  target_compile_definitions(mytarget PUBLIC DEBUG_BUILD)
endif()
```

Another example would be to tell the compiler the result of the previous `HAS_FOOBAR_CALL` test that was discussed earlier in this chapter. You could do this with the following:
```cmake
if (HAS_FOOBAR_CALL)
  target_compile_definitions(mytarget PUBLIC HAS_FOOBAR_CALL)
endif()
```

## How to Configure a Header File
The second approach for passing definitions to the source code is to configure a header file. The header file will include all of the `#define` macros needed to build the project. To configure a file with CMake, the [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file) command is used. This command requires an input file that is parsed by CMake to produce an output file with all variables expanded or replaced. There are three ways to specify a variable in an input file for [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file).
```cmake
#cmakedefine VARIABLE
```

If VARIABLE is true, then the result will be:
```cmake
#define VARIABLE
```

If VARIABLE is false, then the result will be:
```cmake
/* #undef VARIABLE */
```

When writing a file to be configured, consider using `@VARIABLE@` instead of `${VARIABLE}` for variables that are expected to be expanded by CMake. Since the `${}` syntax is commonly used by other languages, users can tell the [`configure_file`](https://cmake.org/cmake/help/latest/command/configure_file.html#command:configure_file) command to only expand variables using the `@var@` syntax by passing the `@ONLY` option to the command; this is useful if you are configuring a script that may contain `${var}` strings that you want to preserve. This is important because CMake will replace all occurrences of `${var}` with the empty string if `var` is not defined in CMake.

The following example configures a .h file for a project that contains preprocessor variables. The first definition indicates if the `FOOBAR` call exists in the library, and the next one contains the path to the build tree.

```cmake
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

```cmake
// -----projectConfigure.h.in file------
/* define a variable to tell the code if the */
/* foobar call is available on this system */
#cmakedefine HAS_FOOBAR_CALL

/* define a variable with the path to the */
/* build directory  */
#define PROJECT_BINARY_DIR "@PROJECT_BINARY_DIR@"
```

It is important to configure files into the binary tree, not the source tree. A single source tree may be shared by multiple build trees or platforms. By configuring files into the binary tree the differences between builds or platforms will be kept isolated in the build tree and will not corrupt other builds. This means that you will need to include the directory of the build tree where you configured the header file into the project’s list of include directories using the [`target_include_directories`](https://cmake.org/cmake/help/latest/command/target_include_directories.html#command:target_include_directories) command.
