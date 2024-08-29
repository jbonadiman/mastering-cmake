# Checking Versions of CMake
CMake is an evolving program and as new versions are released, new features or commands are introduced. As a result, there may be instances where you might want to use a command that is in a current version of CMake but not in previous versions. There are a couple of ways to handle this; one option is to use the if command to check whether a new command exists. For example:
```cmake
# test if the command exists
if(COMMAND some_new_command)
  # use the command
  some_new_command( ARGS...)
endif()
```

Alternatively, one may test against the actual version of CMake that is being run by evaluating the [`CMAKE_VERSION`](https://cmake.org/cmake/help/latest/variable/CMAKE_VERSION.html#variable:CMAKE_VERSION) variable:
```cmake
# look for newer versions of CMake
if(${CMAKE_VERSION} VERSION_GREATER 3.20)
  # do something special here
endif()
```

Finally, some new releases of CMake might no longer support some behavior you were using (although we try to avoid this). In these cases, use CMake policies, as discussed in the [`cmake-policies`](https://cmake.org/cmake/help/latest/manual/cmake-policies.7.html#manual:cmake-policies(7)) manual.
