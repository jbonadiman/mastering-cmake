# The Policy Stack
Policy settings are scoped using a stack. A new level of the stack is pushed when entering a new subdirectory of the project (with [`add_subdirectory`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory)) and popped when leaving it. Therefore, setting a policy in one directory of a project will not affect parent or sibling directories, but it will affect subdirectories.

This is useful when a project contains subprojects that are maintained separately yet built inside the tree. The top-level CMakeLists file in a project may write
```sh
cmake_policy(VERSION 2.6)
project(MyProject)
add_subdirectory(OtherProject)
# ... code requiring new behavior as of CMake 2.6 ...
```

while the `OtherProject/CMakeLists.txt` file contains
```sh
cmake_policy(VERSION 2.4)
projectS(OtherProject)
# ... code that builds with CMake 2.4 ...
```

This allows a project to be updated to CMake 2.6 while subprojects, modules, and included files continue to build with CMake 2.4 until their maintainers update them.

User code may use the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command to push and pop its own stack levels as long as every push is paired with a pop. This is useful when temporarily requesting different behavior for a small section of code. For example, policy [`CMP0003`](https://cmake.org/cmake/help/latest/policy/CMP0003.html#policy:CMP0003) removes extra link directories that used to be included when new behavior is used. When incrementally updating a project, it may be difficult to build a particular target with the remaining targets being OK. The code
```sh
cmake_policy(PUSH)
cmake_policy(SET CMP0003 OLD) # use old-style link for now
add_executable(myexe ...)
cmake_policy(POP)
```

will silence the warning and use the old behavior for that target. You can get a list of policies and help on specific policies by running CMake from the command line as follows
```sh
cmake --help-command cmake_policy
cmake --help-policies
cmake --help-policy CMP0003
```
