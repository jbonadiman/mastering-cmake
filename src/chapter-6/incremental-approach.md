# The Incremental Approach
Another approach to updating a project for a new version of CMake is to deal with each warning one-by-one. One advantage of this approach is that the project will continue to build throughout the process, so the changes can be made incrementally.

When CMake encounters a situation where it needs to know whether to use the old or new behavior for a policy, it checks whether the project has set the policy. If the policy is set, CMake silently uses the corresponding behavior. If the policy is not set, CMake uses the old behavior but warns the author that the policy is not set.

In many cases, a warning message will point to the exact line of code in the CMakeLists files that caused the warning. In some cases, the situation cannot be diagnosed until CMake is generating the native build system rules for the project, so the warning will not include explicit context information. In these cases, CMake will try to provide some information about where code may need to be changed. The documentation for these “generation-time” policies should indicate the point in the project code where the policy should be set to take effect.

In order to incrementally update a project, one warning should be addressed at a time. Several cases may occur, as described below.
