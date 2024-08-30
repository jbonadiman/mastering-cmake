# Policies
Occasionally a new feature or change is made to CMake that is not fully backwards compatible with older versions. This can create problems when someone tries to use an old CMakeLists file with a new version of CMake. To help both end users and developers through such issues, we have introduced [`cmake-policies`](https://cmake.org/cmake/help/latest/manual/cmake-policies.7.html#manual:cmake-policies(7)). Policies are a mechanism for helping improve backwards compatibility and tracking compatibility issues between different versions of CMake.

## Design Goals
There were four main design goals for the CMake policy mechanism:
1. Existing projects should build with newer versions of CMake than that used by the project authors.
    - Users should not need to edit code to get the projects to build.
    - Warnings may be issued but the projects should build.
2. Correctness of new interfaces or bug fixes in old interfaces should not be inhibited by compatibility requirements. Any reduction in correctness of the latest interface is not fair on new projects.
3. Every change made to CMake that may require changes to a project’s CMakeLists files should be documented.
    - Each change should also have a unique identifier that can be referenced with warning and error messages.
    - The new behavior is enabled only when the project has somehow indicated it is supported.
4. We must be able to eventually remove code that implements compatibility with ancient CMake versions.
    - Such removal is necessary to keep the code clean and to allow for internal refactoring.
    - After such removal, attempts at building projects written for ancient versions must fail with an informative message.

All policies in CMake are assigned a name in the form CMPNNNN where NNNN is an integer value. Policies typically support both an old behavior that preserves compatibility with earlier versions of CMake, and a new behavior that is considered correct and preferred for use by new projects. Every policy has documentation detailing the motivation for the change, and the old and new behaviors.

## Setting Policies
Projects may configure the setting of each policy to request old or new behaviors. When CMake encounters user code that may be affected by a particular policy, it checks to see whether the project has set the policy. If the policy has been set (to `OLD` or `NEW`) then CMake follows the behavior specified. If the policy has not been set then the old behavior is used, but a warning is issued telling the project author to set the policy.

There are a couple ways to set the behavior of a policy. The quickest way is to set all policies to a version that corresponds to the release version of CMake the project was written in. Setting the policy version requests the new behavior for all policies introduced in the corresponding version of CMake or earlier. Policies introduced in later versions are marked as “not set” in order to produce proper warning messages. The policy version is set using the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command’s `VERSION` signature. For example, the code
```cmake
cmake_policy(VERSION 3.20)
```

will request the new behavior for all policies introduced in CMake 3.20 or earlier.

The [`cmake_minimum_required`](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html#command:cmake_minimum_required) command will require a minimum version of CMake and will call [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy). A project should always begin with the lines
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject)
# ...code using CMake 3.20 policies
```

This indicates that the person running CMake must have at least version 3.20. If they are running an older version of CMake, an error message will be displayed telling them that the project requires at least the specified version of CMake.

Of course, one should replace “3.20” with the version of CMake you are currently writing to. You can also set each policy individually if you wish; this is sometimes helpful for project authors who want to incrementally convert their projects to use a new behavior, or silence warnings about dependence on an old behavior. The [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command’s `SET` option may be used to explicitly request old or new behavior for a particular policy.

For example, CMake 2.6 introduced the policy [`CMP0002`](https://cmake.org/cmake/help/latest/policy/CMP0002.html#policy:CMP0002), which requires all logical target names to be globally unique (duplicate target names previously worked by accident in some cases, but were not diagnosed). Projects using duplicate target names and working accidentally will receive warnings referencing the policy. The warnings may be silenced with the code
```cmake
cmake_policy(SET CMP0002 OLD)
```

which explicitly tells CMake to use the old behavior for the policy (silently accepting duplicate target names). Another option is to use the code
```cmake
cmake_policy(SET CMP0002 NEW)
```

to explicitly tell CMake to use new behavior and produce an error when a duplicate target is created. Once this is added to the project, it will not build until the author removes any duplicate target names.

When a new version of CMake is released, it introduces new policies that can still build old projects, because by default they do not request `NEW` behavior for any of the new policies. When starting a new project, one should always specify the most recent release of CMake to be supported with the [`cmake_minimum_required`](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html#command:cmake_minimum_required) command. This will ensure that the project is written to work using policies from that version of CMake and not using any old behavior. If no policy version is set, CMake will warn and assume a policy version of 2.4. This allows existing projects that do not specify [`cmake_minimum_required`](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html#command:cmake_minimum_required) to build as they would have with CMake 2.4.

## The Policy Stack
Policy settings are scoped using a stack. A new level of the stack is pushed when entering a new subdirectory of the project (with [`add_subdirectory`](https://cmake.org/cmake/help/latest/command/add_subdirectory.html#command:add_subdirectory)) and popped when leaving it. Therefore, setting a policy in one directory of a project will not affect parent or sibling directories, but it will affect subdirectories.

This is useful when a project contains subprojects that are maintained separately yet built inside the tree. The top-level CMakeLists file in a project may write
```cmake
cmake_policy(VERSION 2.6)
project(MyProject)
add_subdirectory(OtherProject)
# ... code requiring new behavior as of CMake 2.6 ...
```

while the `OtherProject/CMakeLists.txt` file contains
```cmake
cmake_policy(VERSION 2.4)
projectS(OtherProject)
# ... code that builds with CMake 2.4 ...
```

This allows a project to be updated to CMake 2.6 while subprojects, modules, and included files continue to build with CMake 2.4 until their maintainers update them.

User code may use the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command to push and pop its own stack levels as long as every push is paired with a pop. This is useful when temporarily requesting different behavior for a small section of code. For example, policy [`CMP0003`](https://cmake.org/cmake/help/latest/policy/CMP0003.html#policy:CMP0003) removes extra link directories that used to be included when new behavior is used. When incrementally updating a project, it may be difficult to build a particular target with the remaining targets being OK. The code
```cmake
cmake_policy(PUSH)
cmake_policy(SET CMP0003 OLD) # use old-style link for now
add_executable(myexe ...)
cmake_policy(POP)
```

will silence the warning and use the old behavior for that target. You can get a list of policies and help on specific policies by running CMake from the command line as follows
```cmake
cmake --help-command cmake_policy
cmake --help-policies
cmake --help-policy CMP0003
```

## Updating a Project For a New Version of CMake
When a CMake release introduces new policies, it may generate warnings for some existing projects. These warnings indicate that changes to a project may be necessary for dealing with the new policies. While old releases of a project can continue to build with the warnings, the project development tree should be updated to take the new policies into account. There are two approaches to updating a tree: one-shot and incremental. The question of which one is easier depends on the size of the project and which new policies produce warnings.

### The One-Shot Approach
The simplest approach to updating a project for a new version of CMake is simply to change the policy version which is set at the top of the project. Then, try building with the new CMake version to fix problems. For example, to update a project to build with CMake 3.20, one might write
```cmake
cmake_minimum_required(VERSION 3.20)
```

at the beginning of the top-level CMakeLists file. This tells CMake to use the new behavior for every policy introduced in CMake 3.20 and below. When building this project with CMake 3.20, no warnings will be produced regarding policies because it knows that no policies were introduced in later versions. However, if the project was depending on the old policy behavior, it may not build since CMake is now using the new behavior without warning. It is up to the project author who added the policy version line to fix these issues.

### The Incremental Approach
Another approach to updating a project for a new version of CMake is to deal with each warning one-by-one. One advantage of this approach is that the project will continue to build throughout the process, so the changes can be made incrementally.

When CMake encounters a situation where it needs to know whether to use the old or new behavior for a policy, it checks whether the project has set the policy. If the policy is set, CMake silently uses the corresponding behavior. If the policy is not set, CMake uses the old behavior but warns the author that the policy is not set.

In many cases, a warning message will point to the exact line of code in the CMakeLists files that caused the warning. In some cases, the situation cannot be diagnosed until CMake is generating the native build system rules for the project, so the warning will not include explicit context information. In these cases, CMake will try to provide some information about where code may need to be changed. The documentation for these “generation-time” policies should indicate the point in the project code where the policy should be set to take effect.

In order to incrementally update a project, one warning should be addressed at a time. Several cases may occur, as described below.

### Silence a Warning When the Code is Correct
Many policy warnings may be produced simply because the project has not set the policy even though the project may work correctly with the new behavior (there is no way for CMake to know the difference). For a warning about some policy, `CMP<NNNN>`, you can check whether this is the case by adding
```cmake
cmake_policy(SET CMP<NNNN> NEW)
```

to the top of the project and trying to build it. If the project builds correctly with the new behavior, move on to the next policy warning. If the project does not build correctly, one of the other cases may apply.

### Silence a Warning Without Updating the Code
Users can suppress all instances of a warning `CMP<NNNN>` by adding
```cmake
cmake_policy(SET CMP<NNNN> OLD)
```

to the top of a project. However, we encourage project authors to update their code to work with the new behavior for all policies. This is especially important because versions of CMake in the (distant) future may remove support for old behaviors and produce an error for projects requesting them (which tells the user to get an older versions of CMake to build the project).

### Silence a Warning by Updating Code
When a project does not work correctly with the NEW behaviors for a policy, the code needs to be updated. In order to deal with a warning for some policy `CMP<NNNN>`, add
```cmake
cmake_policy(SET CMP<NNNN> NEW)
```

to the top of the project and then fix the code to work with the NEW behavior.

If many instances of the warning occur fixing all of them simultaneously may be too difficult: instead, a developer may fix them one at a time by using the PUSH/POP signatures of the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command:
```cmake
cmake_policy(PUSH)
cmake_policy(SET CMP<NNNN> NEW)
# ... code updated for new policy behavior ...
cmake_policy(POP)
```

This will request the new behavior for a small region of code that has been fixed. Other instances of the policy warning may still appear and must be fixed separately.

### Updating the Project Policy Version
After addressing all policy warnings and getting the project to build cleanly with the new CMake version one step remains. The policy version set at the top of the project should now be updated to match the new CMake version, just as in the one-shot approach described above. For example, after updating a project to build cleanly with CMake 3.20, users may update the top of the project with the line
```cmake
cmake_minimum_required(VERSION 3.20)
```

This will set all policies introduced in CMake 3.20 or below to use the new behavior. Then users may sweep through the rest of the code and remove the calls that use the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command to request the new behavior incrementally. The end result should look the same as the one-shot approach, but could be attained step-by-step.

### Supporting Multiple CMake Versions
Some projects might want to support a few releases of CMake simultaneously. The goal is to build with an older version, while also working with newer versions without warnings. In order to support both CMake 2.4 and 2.6, one may write code like
```cmake
cmake_minimum_required(VERSION 2.4)
if(COMMAND cmake_policy)
  # policy settings ...
  cmake_policy(SET CMP0003 NEW)
endif()
```

This will set the policies to build with CMake 2.6 and to ignore them for CMake 2.4. In order to support both CMake 2.6 and some policies of CMake 2.8, one may write code like:
```cmake
cmake_minimum_required(VERSION 2.6)
if(POLICY CMP1234)
  # policies not known to CMake 2.6 ...
  cmake_policy(SET CMP1234 NEW)
endif()
```

This will set the policies to build with CMake 2.8 and to ignore them for CMake 2.6. If it is known that the project builds with both CMake 2.6 and CMake 2.8’s new policies users may write:
```cmake
cmake_minimum_required(VERSION 2.6)
if (NOT ${CMAKE_VERSION} VERSION_LESS 2.8)
   cmake_policy(VERSION 2.8)
endif()
```

#### Checking Versions of CMake
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
