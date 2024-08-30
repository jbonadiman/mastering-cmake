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

## Installing Targets
Projects typically install some of the library and executable files created during their build process. The [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command provides the `TARGETS` signature for this purpose.

The `TARGETS` keyword is immediately followed by a list of the targets created using [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) or [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library), which are to be installed. One or more files corresponding to each target will be installed.

Files installed with this signature may be divided into categories such as `ARCHIVE`, `LIBRARY`, or `RUNTIME`. These categories are designed to group target files by typical installation destination. The corresponding keyword arguments are optional, but if present, specify that other arguments following them apply only to target files of that type. Target files are categorized as follows:
- **executables -** `RUNTIME`

    Created by [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) (.exe on Windows, no extension on UNIX)

- **loadable modules -** `LIBRARY`

    Created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) with the `MODULE` option (.dll on Windows, .so on UNIX)

- **shared libraries -** `LIBRARY`

    Created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) with the `SHARED` option on UNIX-like platforms (.so on most UNIX, .dylib on Mac)

- **dynamic-link libraries -** `RUNTIME`

    Created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) with the `SHARED` option on Windows platforms (.dll)

- **import libraries -** `ARCHIVE`

    A linkable file created by a dynamic-link library that exports symbols (.lib on most Windows, .dll.a on Cygwin and MinGW).

- **static libraries -** `ARCHIVE`

    Created by [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) with the `STATIC` option (.lib on Windows, .a on UNIX, Cygwin, and MinGW)

Consider a project that defines an executable, `myExecutable`, which links to a shared library `mySharedLib`. It also provides a static library `myStaticLib` and a plugin module to the executable called `myPlugin` that also links to the shared library. The executable, static library, and plugin file may be installed individually using the commands
```cmake
install(TARGETS myExecutable DESTINATION bin)
install(TARGETS myStaticLib DESTINATION lib/myproject)
install(TARGETS myPlugin DESTINATION lib)
```

The executable will not be able to run from the installed location until the shared library to it links to is also installed. Installation of the library requires a bit more care in order to support all platforms. It must be installed in a location searched by the dynamic linker on each platform. On UNIX-like platforms, the library is typically installed to `lib`, while on Windows it should be placed next to the executable in `bin`. An additional challenge is that the import library associated with the shared library on Windows should be treated like the static library, and installed to `lib/myproject`. In other words, we have three different kinds of files created with a single target name that must be installed to three different destinations! Fortunately, this problem can be solved using the category keyword arguments. The shared library may be installed using the command:
```cmake
install(TARGETS mySharedLib
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/myproject)
```

This tells CMake that the `RUNTIME` file (.dll) should be installed to `bin`, the `LIBRARY` file (.so) should be installed to `lib`, and the `ARCHIVE` (.lib) file should be installed to `lib/myproject`. On UNIX, the `LIBRARY` file will be installed; on Windows, the `RUNTIME` and `ARCHIVE` files will be installed.

If the above sample project is to be packaged into separate run time and development components, we must assign the appropriate component to each target file installed. The executable, shared library, and plugin are required in order to run the application, so they belong in a `Runtime` component. Meanwhile, the import library (corresponding to the shared library on Windows) and the static library are only required to develop extensions to the application, and therefore belong in a `Development` component.

Component assignments may be specified by adding the `COMPONENT` argument to each of the commands above. You may also combine all of the installation rules into a single command invocation, which is equivalent to all of the above commands with components added. The files generated by each target are installed using the rule for their category.
```cmake
install(TARGETS myExecutable mySharedLib myStaticLib myPlugin
        RUNTIME DESTINATION bin           COMPONENT Runtime
        LIBRARY DESTINATION lib           COMPONENT Runtime
        ARCHIVE DESTINATION lib/myproject COMPONENT Development)
```

Either `NAMELINK_ONLY` or `NAMELINK_SKIP` may be specified as a `LIBRARY` option. On some platforms, a versioned shared library has a symbolic link such as
```cmake
lib<name>.so -> lib<name>.so.1
```

where `lib<name>.so.1` is the soname of the library, and `lib<name>.so` is a “namelink” that helps linkers to find the library when given `-l<name>`. The `NAMELINK_ONLY` option results in installation of only the namelink when a library target is installed. The `NAMELINK_SKIP` option causes installation of library files other than the namelink when a library target is installed. When neither option is given, both portions are installed. On platforms where versioned shared libraries do not have namelinks, or when a library is not versioned, the `NAMELINK_SKIP` option installs the library and the `NAMELINK_ONLY` option installs nothing. See the [`VERSION`](https://cmake.org/cmake/help/latest/prop_tgt/VERSION.html#prop_tgt:VERSION) and [`SOVERSION`](https://cmake.org/cmake/help/latest/prop_tgt/SOVERSION.html#prop_tgt:SOVERSION) target properties for details on creating versioned, shared libraries.

## Installing Files
Projects may install files other than those that are created with [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) or [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library), such as header files or documentation. General-purpose installation of files is specified using the `FILES` signature.

The `FILES` keyword is immediately followed by a list of files to be installed. Relative paths are evaluated with respect to the current source directory. Files will be installed to the given `DESTINATION` directory. For example, the command
```cmake
install(FILES my-api.h ${CMAKE_CURRENT_BINARY_DIR}/my-config.h
        DESTINATION include)
```

installs the file `my-api.h` from the source tree, and the file `my-config.h` from the build tree into the include directory under the installation prefix. By default installed files are given the permissions `OWNER_WRITE`, `OWNER_READ`, `GROUP_READ`, and `WORLD_READ`, but this may be overridden by specifying the `PERMISSIONS` option. Consider cases in which users would want to install a global configuration file on a UNIX system that is readable only by its owner (such as root). We accomplish this with the command
```cmake
install(FILES my-rc DESTINATION /etc
        PERMISSIONS OWNER_WRITE OWNER_READ)
```

which installs the file `my-rc` with owner read/write permission into the absolute path `/etc`.

The `RENAME` argument specifies a name for an installed file that may be different from the original file. Renaming is allowed only when a single file is installed by the command. For example, the command
```cmake
install(FILES version.h DESTINATION include RENAME my-version.h)
```

will install the file `version.h` from the source directory to `include/my-version.h` under the installation prefix.

## Installing Programs
Projects may also install helper programs, such as shell scripts or Python scripts that are not actually compiled as targets. These may be installed with the `FILES` signature using the `PERMISSIONS` option to add execute permission. However, this case is common enough to justify a simpler interface. CMake provides the `PROGRAMS` signature for this purpose.

The `PROGRAMS` keyword is immediately followed by a list of scripts to be installed. This command is identical to the `FILES` signature, except that the default permissions additionally include `OWNER_EXECUTE`, `GROUP_EXECUTE`, and `WORLD_EXECUTE`. For example, we may install a Python utility script with the command
```cmake
install(PROGRAMS my-util.py DESTINATION bin)
```

which installs `my-util.py` to the `bin` directory under the installation prefix and gives it owner, group, world read and execute permissions, plus owner write.

## Installing Directories
Projects may also provide an entire directory full of resource files, such as icons or html documentation. An entire directory may be installed using the `DIRECTORY` signature.

The `DIRECTORY` keyword is immediately followed by a list of directories to be installed. Relative paths are evaluated with respect to the current source directory. Each named directory is installed to the destination directory. The last component of each input directory name is appended to the destination directory as that directory is copied. For example, the command
```cmake
install(DIRECTORY data/icons DESTINATION share/myproject)
```

will install the `data/icons` directory from the source tree into `share/myproject/icons` under the installation prefix. A trailing slash will leave the last component empty and install the contents of the input directory to the destination. The command
```cmake
install(DIRECTORY doc/html/ DESTINATION doc/myproject)
```

installs the contents of `doc/html` from the source directory into `doc/myproject` under the installation prefix. If no input directory names are given, as in
```cmake
install(DIRECTORY DESTINATION share/myproject/user)
```

the destination directory will be created but nothing will be installed into it.

Files installed by the `DIRECTORY` signature are given the same default permissions as the `FILES` signature. Directories installed by the `DIRECTORY` signature are given the same default permissions as the `PROGRAMS` signature. The `FILE_PERMISSIONS` and `DIRECTORY_PERMISSIONS` options may be used to override these defaults. Consider a case in which a directory full of example shell scripts is to be installed into a directory that is both owner and group writable. We may use the command
```cmake
install(DIRECTORY data/scripts DESTINATION share/myproject
        FILE_PERMISSIONS
          OWNER_READ OWNER_EXECUTE OWNER_WRITE
          GROUP_READ GROUP_EXECUTE
          WORLD_READ WORLD_EXECUTE
        DIRECTORY_PERMISSIONS
          OWNER_READ OWNER_EXECUTE OWNER_WRITE
          GROUP_READ GROUP_EXECUTE GROUP_WRITE
          WORLD_READ WORLD_EXECUTE
        )
```

which installs the directory `data/scripts` into `share/myproject/scripts` and sets the desired permissions. In some cases, a fully-prepared input directory created by the project may have the desired permissions already set. The `USE_SOURCE_PERMISSIONS` option tells CMake to use the file and directory permissions from the input directory during installation. If in the previous example the input directory were to have already been prepared with correct permissions, the following command may have been used instead:
```cmake
install(DIRECTORY data/scripts DESTINATION share/myproject
        USE_SOURCE_PERMISSIONS)
```

If the input directory to be installed is under source management, there may be extra subdirectories in the input that you do not wish to install. There may also be specific files that should not be installed or be installed with different permissions, while most files get the defaults. The `PATTERN` and `REGEX` options may be used for this purpose. A `PATTERN` option is followed first by a globbing pattern and then by an `EXCLUDE` or `PERMISSIONS` option. A `REGEX` option is followed first by a regular expression and then by `EXCLUDE` or `PERMISSIONS`. The `EXCLUDE` option skips installation of those files or directories matching the preceding pattern or expression, while the `PERMISSIONS` option assigns specific permissions to them.

Each input file and directory is tested against the pattern or regular expression as a full path with forward slashes. A pattern will match only complete file or directory names occurring at the end of the full path, while a regular expression may match any portion. For example, the pattern `foo*` will match `.../foo.txt` but not `.../myfoo`.txt or `.../foo/bar.txt;` however, the regular expression `foo` will match all of them.

Returning to the above example of installing an icons directory, consider the case in which the input directory is managed by git and also contains some extra text files that we do not want to install. The command
```cmake
install(DIRECTORY data/icons DESTINATION share/myproject
        PATTERN ".git" EXCLUDE
        PATTERN "*.txt" EXCLUDE)
```

installs the icons directory while ignoring any .git directory or text file contained. The equivalent command using the `REGEX` option is
```cmake
install(DIRECTORY data/icons DESTINATION share/myproject
        REGEX "/.git$" EXCLUDE
        REGEX "/[^/]*.txt$" EXCLUDE)
```

which uses ‘/’ and ‘$’ to constrain the match in the same way as the patterns. Consider a similar case in which the input directory contains shell scripts and text files that we wish to install with different permissions than the other files. The command
```cmake
install(DIRECTORY data/other/ DESTINATION share/myproject
        PATTERN ".git" EXCLUDE
        PATTERN "*.txt"
          PERMISSIONS OWNER_READ OWNER_WRITE
        PATTERN "*.sh"
          PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE)
```

will install the contents of `data/other` from the source directory to `share/myproject` while ignoring .git directories and giving specific permissions to `.txt` and `.sh` files.

## Installing Scripts
Project installations may need to perform tasks other than just placing files in the installation tree. Third-party packages may provide their own mechanisms for registering new plugins that must be invoked during project installation. The `SCRIPT` signature is provided for this purpose.

The `SCRIPT` keyword is immediately followed by the name of a CMake script. CMake will execute the script during installation. If the file name given is a relative path, it will be evaluated with respect to the current source directory. A simple use case is printing a message during installation. We first write a `message.cmake` file containing the code
```cmake
message("Installing My Project")
```

and then reference this script using the command:
```cmake
install(SCRIPT message.cmake)
```

Custom installation scripts are not executed during the main CMakeLists file processing; they are executed during the installation process itself. Variables and macros defined in the code containing the `install (SCRIPT)` call will not be accessible from the script. However, there are a few variables defined during the script execution that may be used to get information about the installation. The variable [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html#variable:CMAKE_INSTALL_PREFIX) is set to the actual installation prefix. This may be different from the corresponding cache variable value, because the installation scripts may be executed by a packaging tool that uses a different prefix. An environment variable `ENV{DESTDIR}` may be set by the user or packaging tool. Its value is prepended to the installation prefix and to absolute installation paths to determine the location where files are installed. In order to reference an install location on disk, custom script may use `$ENV{DESTDIR}${CMAKE_INSTALL_PREFIX}` as the top portion of the path. The variable `CMAKE_INSTALL_CONFIG_NAME` is set to the name of the build configuration currently being installed (Debug, Release, etc.). During component-specific installation, the variable `CMAKE_INSTALL_COMPONENT` is set to the name of the current component.

## Installing Code
Custom installation scripts, as simple as the message above, are more easily created with the script code placed inline in the call to the [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command. The `CODE` signature is provided for this purpose.

The `CODE` keyword is immediately followed by a string containing the code to place in the installation script. An install-time message may be created using the command
```cmake
install(CODE "MESSAGE(\"Installing My Project\")")
```

which has the same effect as the `message.cmake` script but contains the code inline.

## Installing Prerequisite Shared Libraries
Executables are frequently built using shared libraries as building blocks. When you install such an executable, you must also install its prerequisite shared libraries, called “prerequisites” because the executable requires their presence in order to load and run properly. The three main sources of shared libraries are the operating system itself, the build products of your own project, and third party libraries belonging to an external project. The ones from the operating system may be relied upon to be present without installing anything: they are on the base platform where your executable runs. The build products in your own project presumably have [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library) build rules in the CMakeLists files, and so it should be straightforward to create CMake install rules for them. It is the third party libraries that frequently become a high maintenance item when there are more than a handful of them, or when the set of them fluctuates from version-to-version of the third party project. Libraries may be added, code may be reorganized, and the third party shared libraries themselves may actually have additional prerequisites that are not obvious at first glance.

CMake provides a module, [`BundleUtilities`](https://cmake.org/cmake/help/latest/module/BundleUtilities.html#module:BundleUtilities) to make it easier to deal with required shared libraries. This module provides the `fixup_bundle` function to copy and fix prerequisite shared libraries using well-defined locations relative to the executable. For Mac bundle applications, it embeds the libraries inside the bundle, fixing them with `install_name_tool` to make a self-contained unit. On Windows, it copies the libraries into the same directory with the executable since executables will search in their own directories for their required DLLs.

The `fixup_bundle` function helps you create relocatable install trees. Mac users appreciate self-contained bundle applications: you can drag them anywhere, double click them, and they still work. They do not rely on anything being installed in a certain location other than the operating system itself. Similarly, Windows users without administrative privileges appreciate a relocatable install tree where an executable and all required DLLs are installed in the same directory, so that it works no matter where you install it. You can even move things around after installing them and it will still work.

To use `fixup_bundle`, first install one of your executable targets. Then, configure a CMake script that can be called at install time. Inside the configured CMake script, simply [`include`](https://cmake.org/cmake/help/latest/command/include.html#command:include) [`BundleUtilities`](https://cmake.org/cmake/help/latest/module/BundleUtilities.html#module:BundleUtilities) and call the `fixup_bundle` function with appropriate arguments.

In CMakeLists.txt:
```cmake
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
```cmake
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
