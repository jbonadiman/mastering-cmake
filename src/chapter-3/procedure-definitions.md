# Procedure Definitions
The [`macro`](https://cmake.org/cmake/help/latest/command/macro.html#command:macro) and [`function`](https://cmake.org/cmake/help/latest/command/function.html#command:function) commands support repetitive tasks that may be scattered throughout your CMakeLists files. Once a macro or function is defined, it can be used by any CMakeLists files processed after its definition.

A function in CMake is very much like a function in C or C++. You can pass arguments into it, and they become variables within the function. Likewise, some standard variables such as `ARGC`, `ARGV`, `ARGN`, and `ARGV0`, `ARGV1`, etc. are defined. Function calls have a dynamic scope. Within a function you are in a new variable scope; this is like how you drop into a subdirectory using the [`add_subdirectory`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory) command and are in a new variable scope. All the variables that were defined when the function was called remain defined, but any changes to variables or new variables only exist within the function. When the function returns, those variables will go away. Put more simply: when you invoke a function, a new variable scope is pushed; when it returns, that variable scope is popped.

The [`function`](https://cmake.org/cmake/help/latest/command/function.html#command:function) command defines a new function. The first argument is the name of the function to define; all additional arguments are formal parameters to the function.
```sh
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
```sh
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
```sh
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
