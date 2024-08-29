# Installing Scripts
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
