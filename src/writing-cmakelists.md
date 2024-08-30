# Writing CMakeLists Files
This chapter will cover the basics of writing effective CMakeLists files for your software. It will cover the basic commands and issues you will need to handle most projects. While CMake can handle extremely complex projects, for most projects you will find this chapter’s contents will tell you all you need to know. CMake is driven by the CMakeLists.txt files written for a software project. The CMakeLists files determine everything from which options to present to users, to which source files to compile. In addition to discussing how to write a CMakeLists file, this chapter will also cover how to make them robust and maintainable.

## Editing CMakeLists Files
CMakeLists files can be edited in almost any text editor. Some editors, such as Notepad++, come with CMake syntax highlighting and indentation support built-in. For editors such as Emacs or Vim, CMake includes indentation and syntax highlighting modes. These can be found in the `Auxiliary` directory of the source distribution, or downloaded from the CMake [Download](https://cmake.org/cmake/help/book/mastering-cmake/chapter/www.cmake.org/download) page.

Within any of the supported generators (Makefiles, Visual Studio, etc.), if you edit a CMakeLists file and rebuild, there are rules that will automatically invoke CMake to update the generated files (e.g. Makefiles or project files) as required. This helps to assure that your generated files are always in sync with your CMakeLists files.

## CMake Language
The CMake language is composed of comments, commands, and variables.

## Comments
Comments start with `#` and run to the end of the line. See the [`cmake-language`](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#manual:cmake-language(7)) manual for more details.

## Variables
CMakeLists files use variables much like any programming language. CMake variable names are case sensitive and may only contain alphanumeric characters and underscores.

A number of useful variables are automatically defined by CMake and are discussed in the [`cmake-variables`](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html#manual:cmake-variables(7)) manual. These variables begin with `CMAKE_`. Avoid this naming convention (and, ideally, establish your own) for variables specific to your project.

All CMake variables are stored internally as strings although they may sometimes be interpreted as other types.

Use the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command to set variable values. In its simplest form, the first argument to [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) is the name of the variable and the rest of the arguments are the values. Multiple value arguments are packed into a semicolon-separated list and stored in the variable as a string. For example:
```cmake
set(Foo "")      # 1 quoted arg -> value is ""
set(Foo a)       # 1 unquoted arg -> value is "a"
set(Foo "a b c") # 1 quoted arg -> value is "a b c"
set(Foo a b c)   # 3 unquoted args -> value is "a;b;c"
```

Variables may be referenced in command arguments using syntax `${VAR}` where `VAR` is the variable name. If the named variable is not defined, the reference is replaced with an empty string; otherwise it is replaced by the value of the variable. Replacement is performed prior to the expansion of unquoted arguments, so variable values containing semicolons are split into zero-or-more arguments in place of the original unquoted argument. For example:
```cmake
set(Foo a b c)    # 3 unquoted args -> value is "a;b;c"
command(${Foo})   # unquoted arg replaced by a;b;c
                  # and expands to three arguments
command("${Foo}") # quoted arg value is "a;b;c"
set(Foo "")       # 1 quoted arg -> value is empty string
command(${Foo})   # unquoted arg replaced by empty string
                  # and expands to zero arguments
command("${Foo}") # quoted arg value is empty string
```

System environment variables and Windows registry values can be accessed directly in CMake. To access system environment variables, use the syntax `$ENV{VAR}`. CMake can also reference registry entries in many commands using a syntax of the form `[HKEY_CURRENT_USER\\Software\\path1\\path2;key]`, where the paths are built from the registry tree and key.

### Variable Scope
Variables in CMake have a scope that is a little different from most languages. When you set a variable, it is visible to the current CMakeLists file or function and any subdirectory’s CMakeLists files, any functions or macros that are invoked, and any files that are included using the [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) command. When a new subdirectory is processed (or a function called), a new variable scope is created and initialized with the current value of all variables in the calling scope. Any new variables created in the child scope, or changes made to existing variables, will not impact the parent scope. Consider the following example:
```cmake
function(foo)
  message(${test}) # test is 1 here
  set(test 2)
  message(${test}) # test is 2 here, but only in this scope
endfunction()

set(test 1)
foo()
message(${test}) # test will still be 1 here
```

In some cases, you might want a function or subdirectory to set a variable in its parent’s scope. There is a way for CMake to return a value from a function, and it can be done by using the `PARENT_SCOPE` option with the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command. We can modify the prior example so that the function `foo` changes the value of test in its parent’s scope as follows:
```cmake
function(foo)
  message(${test}) # test is 1 here
  set(test 2 PARENT_SCOPE)
  message(${test}) # test still 1 in this scope
endfunction()

set(test 1)
foo()
message(${test}) # test will now be 2 here
```
Variables in CMake are defined in the order of the execution of [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) commands.

Consider the following example:
```cmake
# FOO is undefined

set(FOO 1)
# FOO is now set to 1

set(FOO 0)
# FOO is now set to 0
```

To understand the scope of variables, consider this example:
```cmake
set(foo 1)

# process the dir1 subdirectory
add_subdirectory(dir1)

# include and process the commands in file1.cmake
include(file1.cmake)

set(bar 2)
# process the dir2 subdirectory
add_subdirectory(dir2)

# include and process the commands in file2.cmake
include(file2.cmake)
```

In this example, because the variable `foo` is defined at the beginning, it will be defined while processing both dir1 and dir2. In contrast, `bar` will only be defined when processing dir2. Likewise, `foo` will be defined when processing both file1.cmake and file2.cmake, whereas `bar` will only be defined while processing file2.cmake.

## Commands
A command consists of the command name, opening parenthesis, whitespace separated arguments, and a closing parenthesis. Each command is evaluated in the order that it appears in the CMakeLists file. See the [`cmake-commands`](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html#manual:cmake-commands(7)) manual for a full list of CMake commands.

CMake is no longer case sensitive to command names, so where you see `command`, you could use `COMMAND` or `Command` instead. It is considered best practice to use lowercase commands. All whitespace (spaces, line feeds, tabs) is ignored except to separate arguments. Therefore, commands may span multiple lines as long as the command name and the opening parenthesis are on the same line.

CMake command arguments are space separated and case sensitive. Command arguments may be either quoted or unquoted. A quoted argument starts and ends in a double quote (“) and always represents exactly one argument. Any double quotes contained inside the value must be escaped with a backslash. Consider using bracket arguments for arguments that require escaping, see the [`cmake-language`](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#manual:cmake-language(7)) manual. An unquoted argument starts in any character other than a double quote (later double quotes are literal) and is automatically expanded into zero-or-more arguments by separating on semicolons within the value. For example:
```cmake
command("")          # 1 quoted argument
command("a b c")     # 1 quoted argument
command("a;b;c")     # 1 quoted argument
command("a" "b" "c") # 3 quoted arguments
command(a b c)       # 3 unquoted arguments
command(a;b;c)       # 1 unquoted argument expands to 3
```

### Basic Commands
As we saw earlier, the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) and [`unset`](https://cmake.org/cmake/help/latest/command/unset.html#command:unset) commands explicitly set or unset variables. The [`string`](https://cmake.org/cmake/help/latest/command/string.html#command:string), [`list`](https://cmake.org/cmake/help/latest/command/list.html#command:list), and [`separate_arguments`](https://cmake.org/cmake/help/latest/command/separate_arguments.html#command:separate_arguments) commands offer basic manipulation of strings and lists.

The [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) and [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) commands are the main commands for defining the executables and libraries to build, and which source files comprise them. For Visual Studio projects, the source files will show up in the IDE as usual, but any header files the project uses will not be. To have the header files show up, simply add them to the list of source files for the executable or library; this can be done for all generators. Any generators that do not use the header files directly (such as Makefile based generators) will simply ignore them.

## Flow Control
The CMake language provides three flow control constructs to help organize your CMakeLists files and keep them maintainable.

- Conditional statements (e.g. [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if))
- Looping constructs (e.g. [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) and [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while))
- Procedure definitions (e.g. [`macro`](https://cmake.org/cmake/help/latest/command/macro.html#command:macro) and [`function`](https://cmake.org/cmake/help/latest/command/function.html#command:function))

### Conditional Statements
First we will consider the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command. In many ways, the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command in CMake is just like the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command in any other language. It evaluates its expression and uses it to execute the code in its body or optionally the code in the [`else`](https://cmake.org/cmake/help/latest/command/else.html#command:else) clause. For example:
```cmake
if(FOO)
  # do something here
else()
  # do something else
endif()
```

CMake also supports [`elseif`](https://cmake.org/cmake/help/latest/command/elseif.html#command:elseif) to help sequentially test for multiple conditions. For example:
```cmake
if(MSVC80)
  # do something here
elseif(MSVC90)
  # do something else
elseif(APPLE)
  # do something else
endif()
```

The [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command documents the many conditions it can test.

### Looping Constructs
The [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) and [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) commands allow you to handle repetitive tasks that occur in sequence. The [`break`](https://cmake.org/cmake/help/latest/command/break.html#command:break) command breaks out of a [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) or [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) loop before it would normally end.

The [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) command enables you to execute a group of CMake commands repeatedly on the members of a list. Consider the following example adapted from VTK.
```cmake
foreach(tfile
        TestAnisotropicDiffusion2D
        TestButterworthLowPass
        TestButterworthHighPass
        TestCityBlockDistance
        TestConvolve
        )
  add_test(${tfile}-image ${VTK_EXECUTABLE}
    ${VTK_SOURCE_DIR}/Tests/rtImageTest.tcl
    ${VTK_SOURCE_DIR}/Tests/${tfile}.tcl
    -D ${VTK_DATA_ROOT}
    -V Baseline/Imaging/${tfile}.png
    -A ${VTK_SOURCE_DIR}/Wrapping/Tcl
    )
endforeach()
```

The first argument of the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) command is the name of the variable that will take on a different value with each iteration of the loop; the remaining arguments are the list of values over which to loop. In this example, the body of the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop is just one CMake command, [`add_test`](https://cmake.org/cmake/help/latest/command/add_test.html#command:add_test). In the body of the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach), each time the loop variable (`tfile` in this example) is referenced will be replaced with the current value from the list. In the first iteration, occurrences of `${tfile}` will be replaced with `TestAnisotropicDiffusion2D`. In the next iteration, `${tfile}` will be replaced with `TestButterworthLowPass`. The [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop will continue to loop until all of the arguments have been processed.

It is worth mentioning that [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loops can be nested, and that the loop variable is replaced prior to any other variable expansion. This means that in the body of a [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop, you can construct variable names using the loop variable. In the code below, the loop variable `tfile` is expanded, and then concatenated with `_TEST_RESULT`. The new variable name is then expanded and tested to see if it matches `FAILED`.
```cmake
if(${${tfile}_TEST_RESULT} MATCHES FAILED)
  message("Test ${tfile} failed.")
endif()
```

The [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) command provides looping based on a test condition. The format for the test expression in the [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) command is the same as it is for the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command, as described earlier. Consider the following example, which is used by CTest. Note that CTest updates the value of `CTEST_ELAPSED_TIME` internally.
```cmake
#####################################################
# run paraview and ctest test dashboards for 6 hours
#
while(${CTEST_ELAPSED_TIME} LESS 36000)
  set(START_TIME ${CTEST_ELAPSED_TIME})
  ctest_run_script("dash1_ParaView_vs71continuous.cmake")
  ctest_run_script("dash1_cmake_vs71continuous.cmake")
endwhile()
```

### Procedure Definitions
The [`macro`](https://cmake.org/cmake/help/latest/command/macro.html#command:macro) and [`function`](https://cmake.org/cmake/help/latest/command/function.html#command:function) commands support repetitive tasks that may be scattered throughout your CMakeLists files. Once a macro or function is defined, it can be used by any CMakeLists files processed after its definition.

A function in CMake is very much like a function in C or C++. You can pass arguments into it, and they become variables within the function. Likewise, some standard variables such as `ARGC`, `ARGV`, `ARGN`, and `ARGV0`, `ARGV1`, etc. are defined. Function calls have a dynamic scope. Within a function you are in a new variable scope; this is like how you drop into a subdirectory using the [`add_subdirectory`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory) command and are in a new variable scope. All the variables that were defined when the function was called remain defined, but any changes to variables or new variables only exist within the function. When the function returns, those variables will go away. Put more simply: when you invoke a function, a new variable scope is pushed; when it returns, that variable scope is popped.

The [`function`](https://cmake.org/cmake/help/latest/command/function.html#command:function) command defines a new function. The first argument is the name of the function to define; all additional arguments are formal parameters to the function.
```cmake
function(DetermineTime _time)
  # pass the result up to whatever invoked this
  set(${_time} "1:23:45" PARENT_SCOPE)
endfunction()

# now use the function we just defined
DetermineTime(current_time)

if(DEFINED current_time)
  message(STATUS "The time is now: ${current_time}")
endif()
```

Note that in this example, `_time` is used to pass the name of the return variable. The [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command is invoked with the value of `_time`, which will be `current_time`. Finally, the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command uses the `PARENT_SCOPE` option to set the variable in the caller’s scope instead of the local scope.

Macros are defined and called in the same manner as functions. The main differences are that a macro does not push and pop a new variable scope, and that the arguments to a macro are not treated as variables but as strings replaced prior to execution. This is very much like the differences between a macro and a function in C or C++. The first argument is the name of the macro to create; all additional arguments are formal parameters to the macro.
```cmake
# define a simple macro
macro(assert TEST COMMENT)
  if(NOT ${TEST})
    message("Assertion failed: ${COMMENT}")
  endif()
endmacro()

# use the macro
find_library(FOO_LIB foo /usr/local/lib)
assert(${FOO_LIB} "Unable to find library foo")
```

The simple example above creates a macro called `assert`. The macro is defined into two arguments; the first is a value to test and the second is a comment to print out if the test fails. The body of the macro is a simple [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command with a [`message`](https://cmake.org/cmake/help/latest/command/message.html#command:message) command inside of it. The macro body ends when the [`endmacro`](https://cmake.org/cmake/help/latest/command/endmacro.html#command:endmacro) command is found. The macro can be invoked simply by using its name as if it were a command. In the above example, if `FOO_LIB` was not found then a message would be displayed indicating the error condition.

The [`macro`](https://cmake.org/cmake/help/latest/command/macro.html#command:macro) command also supports defining macros that take variable argument lists. This can be useful if you want to define a macro that has optional arguments or multiple signatures. Variable arguments can be referenced using `ARGC` and `ARGV0`, `ARGV1`, etc., instead of the formal parameters. `ARGV0` represents the first argument to the macro; `ARGV1` represents the next, and so forth. You can also use a mixture of formal arguments and variable arguments, as shown in the example below.
```cmake
# define a macro that takes at least two arguments
# (the formal arguments) plus an optional third argument
macro(assert TEST COMMENT)
  if(NOT ${TEST})
    message("Assertion failed: ${COMMENT}")

    # if called with three arguments then also write the
    # message to a file specified as the third argument
    if(${ARGC} MATCHES 3)
      file(APPEND ${ARGV2} "Assertion failed: ${COMMENT}")
    endif()

  endif()
endmacro()

# use the macro
find_library(FOO_LIB foo /usr/local/lib)
assert(${FOO_LIB} "Unable to find library foo")
```

In this example, the two required arguments are `TEST` and `COMMENT`. These required arguments can be referenced by name, as they are in this example, or by referencing `ARGV0` and `ARGV1`. If you want to process the arguments as a list, use the `ARGV` and `ARGN` variables. `ARGV` (as opposed to `ARGV0`, `ARGV1`, etc.) is a list of all the arguments to the macro, while `ARGN` is a list of all the arguments after the formal arguments. Inside your macro, you can use the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) command to iterate over `ARGV` or `ARGN` as desired.

The [`return`](https://cmake.org/cmake/help/latest/command/return.html#command:return) command returns from a function, directory or file. Note that a macro, unlike a function, is expanded in place and therefore cannot handle [`return`](https://cmake.org/cmake/help/latest/command/return.html#command:return).

## Regular Expressions
A few CMake commands, such as [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) and [`string`](https://cmake.org/cmake/help/latest/command/string.html#command:string), make use of regular expressions or can take a regular expression as an argument. In its simplest form, a regular expression is a sequence of characters used to search for exact character matches. However, many times the exact sequence to be found is unknown, or only a match at the beginning or end of a string is desired. Since there are several different conventions for specifying regular expressions, CMake’s standard is described in the [`string`](https://cmake.org/cmake/help/latest/command/string.html#command:string) command documentation. The description is based on the open source regular expression class from Texas Instruments, which is used by CMake for parsing regular expressions.

## Advanced Commands
There are a few commands that can be very useful, but are not typically used in writing CMakeLists files. This section will discuss a few of these commands and when they are useful.

First, consider the [`add_dependencies`](https://cmake.org/cmake/help/latest/command/add_dependencies.html#command:add_dependencies) command which creates a dependency between two targets. CMake automatically creates dependencies between targets when it can determine them. For example, CMake will automatically create a dependency for an executable target that depends on a library target. The [`add_dependencies`](https://cmake.org/cmake/help/latest/command/add_dependencies.html#command:add_dependencies) command is typically used to specify inter-target dependencies between targets where at least one of the targets is a custom target (see [Add Custom Command](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Custom%20Commands.html#add-custom-command) section).

The [`include_regular_expression`](https://cmake.org/cmake/help/latest/command/include_regular_expression.html#command:include_regular_expression) command also relates to dependencies. This command controls the regular expression that is used for tracing source code dependencies. By default, CMake will trace all the dependencies for a source file including system files such as stdio.h. If you specify a regular expression with the [`include_regular_expression`](https://cmake.org/cmake/help/latest/command/include_regular_expression.html#command:include_regular_expression) command, that regular expression will be used to limit which include files are processed. For example; if your software project’s include files all started with the prefix foo (e.g. `fooMain.c` `fooStruct.h`, etc.), you could specify a regular expression of `^foo.*$` to limit the dependency checking to just the files of your project.
