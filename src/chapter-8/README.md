# Installing Files
Software is typically installed into a directory separate from the source and build trees. This allows it to be distributed in a clean form and isolates users from the details of the build process. CMake provides the [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command to specify how a project is to be installed. This command is invoked by a project in the CMakeLists file and tells CMake how to generate installation scripts. The scripts are executed at install time to perform the actual installation of files. For Makefile generators (UNIX, NMake, MinGW, etc.), the user simply runs `make install` (or `nmake install`) and the make tool will invoke CMake’s installation module. With GUI based systems (Visual Studio, Xcode, etc.), the user simply builds the target called `INSTALL`.

Each call to the [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command defines some installation rules. Within one CMakeLists file (source directory), these rules will be evaluated in the order that the corresponding commands are invoked. The order across multiple directories changed in CMake 3.14.

The [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command has several signatures designed for common installation use cases. A particular invocation of the command specifies the signature as the first argument. The signatures are `TARGETS`, `FILES` or `PROGRAMS`, `DIRECTORY`, `SCRIPT`, `CODE` and `EXPORT`.

- **install(TARGETS…)**

    Installs the binary files corresponding to targets built inside the project.

- **install(FILES…)**

    General-purpose file installation, which is typically used for header files, documentation, and data files required by your software.

- **install(PROGRAMS…)**

    Installs executable files not built by the project, such as shell scripts. This argument is identical to `install(FILES)` except that the default permissions of the installed file include the executable bit.`

- **install(DIRECTORY…)**

    This argument installs an entire directory tree. It may be used for installing directories with resources, such as icons and images.

- **install(SCRIPT…)**

    Specifies a user-provided CMake script file to be executed during installation. This is typically used to define pre-install or post-install actions for other rules.

- **install(CODE…)**

    Specifies user-provided CMake code to be executed during the installation. This is similar to `install (SCRIPT)` but the code is provided inline in the call as a string.

- **install(EXPORT…)**

    Generates and installs a CMake file containing code to import targets from the installation tree into another project.

The `TARGETS`, `FILES`, `PROGRAMS`, and `DIRECTORY` signatures are all meant to create install rules for files. The targets, files, or directories to be installed are listed immediately after the signature name argument. Additional details can be specified using keyword arguments followed by corresponding values. Keyword arguments provided by most of the signatures are as follows.

- **DESTINATION**

    This argument specifies the location where the installation rule will place files, and must be followed by a directory path indicating the location. If the directory is specified as a full path, it will be evaluated at install time as an absolute path. If the directory is specified as a relative path, it will be evaluated at install time relative to the installation prefix. The prefix may be set by the user through the cache variable [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX). A platform-specific default is provided by CMake: `/usr/local` on UNIX, and “\<SystemDrive\>/`Program Files`/\<ProjectName\>” on Windows, where SystemDrive is along the lines of `C:` and ProjectName is the name given to the top-most [`project`](https://cmake.org/cmake/help/latest/command/project.html#command:project) command.

- **PERMISSIONS**

    This argument specifies file permissions to be set on the installed files. This option is needed only to override the default permissions selected by a particular [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command signature. Valid permissions are `OWNER_READ`, `OWNER_WRITE`, `OWNER_EXECUTE`, `GROUP_READ`, `GROUP_WRITE`, `GROUP_EXECUTE`, `WORLD_READ`, `WORLD_WRITE`, `WORLD_EXECUTE`, `SETUID`, and `SETGID`. Some platforms do not support all of these permissions; on such platforms those permission names are ignored.

- **CONFIGURATIONS**

    This argument specifies a list of build configurations for which an installation rule applies (Debug, Release, etc.). For Makefile generators, the build configuration is specified by the [`CMAKE_BUILD_TYPE`](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html#variable:CMAKE_BUILD_TYPE) cache variable. For Visual Studio and Xcode generators, the configuration is selected when the [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) target is built. An installation rule will be evaluated only if the current install configuration matches an entry in the list provided to this argument. Configuration name comparison is case-insensitive.

- **COMPONENT**

    This argument specifies the installation component for which the installation rule applies. Some projects divide their installations into multiple components for separate packaging. For example, a project may define a `Runtime` component that contains the files needed to run a tool; a `Development` component containing the files needed to build extensions to the tool; and a `Documentation` component containing the manual pages and other help files. The project may then package each component separately for distribution by installing only one component at a time. By default, all components are installed. Component-specific installation is an advanced feature intended for use by package maintainers. It requires manual invocation of the installation scripts with an argument defining the `COMPONENT` variable to name the desired component. Note that component names are not defined by CMake. Each project may define its own set of components.

- **OPTIONAL**

    This argument specifies that it is not an error if the input file to be installed does not exist. If the input file exists, it will be installed as requested. If it does not exist, it will be silently not installed.
