# Object Libraries
Large projects often organize their source files into groups, perhaps in separate subdirectories, that each need different include directories and preprocessor definitions. For this use case CMake has developed the concept of Object Libraries.

An Object Library is a collection of source files compiled into an object file which is not linked into a library file or made into an archive. Instead other targets created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) or [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) may reference the objects using an expression of the form `$<TARGET_OBJECTS:name>` as a source, where “name” is the target created by the [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) call. For example:
```cmake
add_library(A OBJECT a.cpp)
add_library(B OBJECT b.cpp)
add_library(Combined $<TARGET_OBJECTS:A> $<TARGET_OBJECTS:B>)
```

will include A and B object files in a library called Combined. Object libraries may contain only sources (and headers) that compile to object files.
