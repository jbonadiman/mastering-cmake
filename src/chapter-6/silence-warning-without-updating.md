# Silence a Warning Without Updating the Code
Users can suppress all instances of a warning `CMP<NNNN>` by adding
```sh
cmake_policy(SET CMP<NNNN> OLD)
```

to the top of a project. However, we encourage project authors to update their code to work with the new behavior for all policies. This is especially important because versions of CMake in the (distant) future may remove support for old behaviors and produce an error for projects requesting them (which tells the user to get an older versions of CMake to build the project).
