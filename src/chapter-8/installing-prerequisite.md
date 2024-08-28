# Installing Prerequisite Shared Libraries
Executables are frequently built using shared libraries as building blocks. When you install such an executable, you must also install its prerequisite shared libraries, called “prerequisites” because the executable requires their presence in order to load and run properly. The three main sources of shared libraries are the operating system itself, the build products of your own project, and third party libraries belonging to an external project. The ones from the operating system may be relied upon to be present without installing anything: they are on the base platform where your executable runs. The build products in your own project presumably have [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) build rules in the CMakeLists files, and so it should be straightforward to create CMake install rules for them. It is the third party libraries that frequently become a high maintenance item when there are more than a handful of them, or when the set of them fluctuates from version-to-version of the third party project. Libraries may be added, code may be reorganized, and the third party shared libraries themselves may actually have additional prerequisites that are not obvious at first glance.

CMake provides a module, [`BundleUtilities`](https://cmake.org/cmake/help/latest/module/BundleUtilities.html#module:BundleUtilities) to make it easier to deal with required shared libraries. This module provides the `fixup_bundle` function to copy and fix prerequisite shared libraries using well-defined locations relative to the executable. For Mac bundle applications, it embeds the libraries inside the bundle, fixing them with `install_name_tool` to make a self-contained unit. On Windows, it copies the libraries into the same directory with the executable since executables will search in their own directories for their required DLLs.

The `fixup_bundle` function helps you create relocatable install trees. Mac users appreciate self-contained bundle applications: you can drag them anywhere, double click them, and they still work. They do not rely on anything being installed in a certain location other than the operating system itself. Similarly, Windows users without administrative privileges appreciate a relocatable install tree where an executable and all required DLLs are installed in the same directory, so that it works no matter where you install it. You can even move things around after installing them and it will still work.

To use `fixup_bundle`, first install one of your executable targets. Then, configure a CMake script that can be called at install time. Inside the configured CMake script, simply [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) [`BundleUtilities`](https://cmake.org/cmake/help/latest/module/BundleUtilities.html#module:BundleUtilities) and call the `fixup_bundle` function with appropriate arguments.

In CMakeLists.txt:
```sh
install(TARGETS myExecutable DESTINATION bin)

# To install, for example, MSVC runtime libraries:
include(InstallRequiredSystemLibraries)

# To install other/non-system 3rd party required libraries:
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/FixBundle.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/FixBundle.cmake
  @ONLY
  )

install(SCRIPT ${CMAKE_CURRENT_BINARY_DIR}/FixBundle.cmake)
```

In FixBundle.cmake.in:
```sh
include(BundleUtilities)

# Set bundle to the full path name of the executable already
# existing in the install tree:
set(bundle
   "${CMAKE_INSTALL_PREFIX}/myExecutable@CMAKE_EXECUTABLE_SUFFIX@")

# Set other_libs to a list of full path names to additional
# libraries that cannot be reached by dependency analysis.
# (Dynamically loaded PlugIns, for example.)
set(other_libs "")

# Set dirs to a list of directories where prerequisite libraries
# may be found:
set(dirs
   "@CMAKE_RUNTIME_OUTPUT_DIRECTORY@"
   "@CMAKE_LIBRARY_OUTPUT_DIRECTORY@"
   )

fixup_bundle("${bundle}" "${other_libs}" "${dirs}")
```

You are responsible for verifying that you have permission to copy and distribute the prerequisite shared libraries for your executable. Some libraries may have restrictive software licenses that prohibit making copies a la `fixup_bundle`.
