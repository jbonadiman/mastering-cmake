# Design Goals
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
