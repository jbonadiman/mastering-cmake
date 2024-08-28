# Variable Scope
Variables in CMake have a scope that is a little different from most languages. When you set a variable, it is visible to the current CMakeLists file or function and any subdirectory’s CMakeLists files, any functions or macros that are invoked, and any files that are included using the [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) command. When a new subdirectory is processed (or a function called), a new variable scope is created and initialized with the current value of all variables in the calling scope. Any new variables created in the child scope, or changes made to existing variables, will not impact the parent scope. Consider the following example:
```sh
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
```sh
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
```sh
# FOO is undefined

set(FOO 1)
# FOO is now set to 1

set(FOO 0)
# FOO is now set to 0
```

To understand the scope of variables, consider this example:
```sh
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
