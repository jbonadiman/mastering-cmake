# Setting Policies
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
