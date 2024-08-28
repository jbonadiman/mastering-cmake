# Variables
CMakeLists files use variables much like any programming language. CMake variable names are case sensitive and may only contain alphanumeric characters and underscores.

A number of useful variables are automatically defined by CMake and are discussed in the [`cmake-variables`](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html#manual:cmake-variables(7)) manual. These variables begin with `CMAKE_`. Avoid this naming convention (and, ideally, establish your own) for variables specific to your project.

All CMake variables are stored internally as strings although they may sometimes be interpreted as other types.

Use the [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) command to set variable values. In its simplest form, the first argument to [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) is the name of the variable and the rest of the arguments are the values. Multiple value arguments are packed into a semicolon-separated list and stored in the variable as a string. For example:
```sh
set(Foo "")      # 1 quoted arg -> value is ""
set(Foo a)       # 1 unquoted arg -> value is "a"
set(Foo "a b c") # 1 quoted arg -> value is "a b c"
set(Foo a b c)   # 3 unquoted args -> value is "a;b;c"
```

Variables may be referenced in command arguments using syntax `${VAR}` where `VAR` is the variable name. If the named variable is not defined, the reference is replaced with an empty string; otherwise it is replaced by the value of the variable. Replacement is performed prior to the expansion of unquoted arguments, so variable values containing semicolons are split into zero-or-more arguments in place of the original unquoted argument. For example:
```sh
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
