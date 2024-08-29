# The One-Shot Approach
The simplest approach to updating a project for a new version of CMake is simply to change the policy version which is set at the top of the project. Then, try building with the new CMake version to fix problems. For example, to update a project to build with CMake 3.20, one might write
```cmake
cmake_minimum_required(VERSION 3.20)
```

at the beginning of the top-level CMakeLists file. This tells CMake to use the new behavior for every policy introduced in CMake 3.20 and below. When building this project with CMake 3.20, no warnings will be produced regarding policies because it knows that no policies were introduced in later versions. However, if the project was depending on the old policy behavior, it may not build since CMake is now using the new behavior without warning. It is up to the project author who added the policy version line to fix these issues.
