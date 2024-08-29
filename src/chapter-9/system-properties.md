# System Properties
Although it is a common practice in C and C++ code to add platform-specific code inside preprocessor `ifdef` directives, for maximum portability this should be avoided. Software should not be tuned to specific platforms with `ifdefs`, but rather to a canonical system consisting of a set of features. Coding to specific systems makes the software less portable, because systems and the features they support change with time, and even from system to system. A feature that may not have worked on a platform in the past may be a required feature for the platform in the future. The following code fragments illustrate the difference between coding to a canonical system and a specific system:
```sh
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
```sh
// --- testNeedFoobar.c -----

#include <foobar.h>
main()
{
  foobar();
}
```

```sh
# --- testNeedFoobar.cmake ---

try_compile (HAS_FOOBAR_CALL
  ${CMAKE_BINARY_DIR}
  ${PROJECT_SOURCE_DIR}/testNeedFoobar.c
  )
```

Now that `HAS_FOOBAR_CALL` is set correctly in CMake, you can use it in your source code through the [`target_compile_definitions`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions) command. Alternatively, it is possible to configure a header file. This is discussed further in the section called [How to Configure a Header File](./configure-header-file.md).

Sometimes compiling a test program is not enough. In some cases, you may actually want to compile and run a program to get its output. A good example of this is testing the byte order of a machine. The following example shows how to write a small program that CMake will compile and run to determine the byte order of a machine.
```sh
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

```sh
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
```sh
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
```sh
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
