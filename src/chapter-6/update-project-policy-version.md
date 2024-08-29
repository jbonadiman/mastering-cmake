# Updating the Project Policy Version
After addressing all policy warnings and getting the project to build cleanly with the new CMake version one step remains. The policy version set at the top of the project should now be updated to match the new CMake version, just as in the one-shot approach described above. For example, after updating a project to build cleanly with CMake 3.20, users may update the top of the project with the line
```cmake
cmake_minimum_required(VERSION 3.20)
```

This will set all policies introduced in CMake 3.20 or below to use the new behavior. Then users may sweep through the rest of the code and remove the calls that use the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command to request the new behavior incrementally. The end result should look the same as the one-shot approach, but could be attained step-by-step.
