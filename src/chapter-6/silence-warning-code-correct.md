# Silence a Warning When the Code is Correct
Many policy warnings may be produced simply because the project has not set the policy even though the project may work correctly with the new behavior (there is no way for CMake to know the difference). For a warning about some policy, `CMP<NNNN>`, you can check whether this is the case by adding
```cmake
cmake_policy(SET CMP<NNNN> NEW)
```

to the top of the project and trying to build it. If the project builds correctly with the new behavior, move on to the next policy warning. If the project does not build correctly, one of the other cases may apply.
