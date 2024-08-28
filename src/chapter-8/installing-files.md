# Installing Files
Projects may install files other than those that are created with [`add_executable`](https://cmake.org/cmake/help/latest/command/add_executable.html#command:add_executable) or [`add_library`](https://cmake.org/cmake/help/latest/command/add_library.html#command:add_library), such as header files or documentation. General-purpose installation of files is specified using the `FILES` signature.

The `FILES` keyword is immediately followed by a list of files to be installed. Relative paths are evaluated with respect to the current source directory. Files will be installed to the given `DESTINATION` directory. For example, the command
```sh
install(FILES my-api.h ${CMAKE_CURRENT_BINARY_DIR}/my-config.h
        DESTINATION include)
```

installs the file `my-api.h` from the source tree, and the file `my-config.h` from the build tree into the include directory under the installation prefix. By default installed files are given the permissions `OWNER_WRITE`, `OWNER_READ`, `GROUP_READ`, and `WORLD_READ`, but this may be overridden by specifying the `PERMISSIONS` option. Consider cases in which users would want to install a global configuration file on a UNIX system that is readable only by its owner (such as root). We accomplish this with the command
```sh
install(FILES my-rc DESTINATION /etc
        PERMISSIONS OWNER_WRITE OWNER_READ)
```

which installs the file `my-rc` with owner read/write permission into the absolute path `/etc`.

The `RENAME` argument specifies a name for an installed file that may be different from the original file. Renaming is allowed only when a single file is installed by the command. For example, the command
```sh
install(FILES version.h DESTINATION include RENAME my-version.h)
```

will install the file `version.h` from the source directory to `include/my-version.h` under the installation prefix.
