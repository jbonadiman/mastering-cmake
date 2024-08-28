# Installing Directories
Projects may also provide an entire directory full of resource files, such as icons or html documentation. An entire directory may be installed using the `DIRECTORY` signature.

The `DIRECTORY` keyword is immediately followed by a list of directories to be installed. Relative paths are evaluated with respect to the current source directory. Each named directory is installed to the destination directory. The last component of each input directory name is appended to the destination directory as that directory is copied. For example, the command
```sh
install(DIRECTORY data/icons DESTINATION share/myproject)
```

will install the `data/icons` directory from the source tree into `share/myproject/icons` under the installation prefix. A trailing slash will leave the last component empty and install the contents of the input directory to the destination. The command
```sh
install(DIRECTORY doc/html/ DESTINATION doc/myproject)
```

installs the contents of `doc/html` from the source directory into `doc/myproject` under the installation prefix. If no input directory names are given, as in
```sh
install(DIRECTORY DESTINATION share/myproject/user)
```

the destination directory will be created but nothing will be installed into it.

Files installed by the `DIRECTORY` signature are given the same default permissions as the `FILES` signature. Directories installed by the `DIRECTORY` signature are given the same default permissions as the `PROGRAMS` signature. The `FILE_PERMISSIONS` and `DIRECTORY_PERMISSIONS` options may be used to override these defaults. Consider a case in which a directory full of example shell scripts is to be installed into a directory that is both owner and group writable. We may use the command
```sh
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
```sh
install(DIRECTORY data/scripts DESTINATION share/myproject
        USE_SOURCE_PERMISSIONS)
```

If the input directory to be installed is under source management, there may be extra subdirectories in the input that you do not wish to install. There may also be specific files that should not be installed or be installed with different permissions, while most files get the defaults. The `PATTERN` and `REGEX` options may be used for this purpose. A `PATTERN` option is followed first by a globbing pattern and then by an `EXCLUDE` or `PERMISSIONS` option. A `REGEX` option is followed first by a regular expression and then by `EXCLUDE` or `PERMISSIONS`. The `EXCLUDE` option skips installation of those files or directories matching the preceding pattern or expression, while the `PERMISSIONS` option assigns specific permissions to them.

Each input file and directory is tested against the pattern or regular expression as a full path with forward slashes. A pattern will match only complete file or directory names occurring at the end of the full path, while a regular expression may match any portion. For example, the pattern `foo*` will match `.../foo.txt` but not `.../myfoo`.txt or `.../foo/bar.txt;` however, the regular expression `foo` will match all of them.

Returning to the above example of installing an icons directory, consider the case in which the input directory is managed by git and also contains some extra text files that we do not want to install. The command
```sh
install(DIRECTORY data/icons DESTINATION share/myproject
        PATTERN ".git" EXCLUDE
        PATTERN "*.txt" EXCLUDE)
```

installs the icons directory while ignoring any .git directory or text file contained. The equivalent command using the `REGEX` option is
```sh
install(DIRECTORY data/icons DESTINATION share/myproject
        REGEX "/.git$" EXCLUDE
        REGEX "/[^/]*.txt$" EXCLUDE)
```

which uses ‘/’ and ‘$’ to constrain the match in the same way as the patterns. Consider a similar case in which the input directory contains shell scripts and text files that we wish to install with different permissions than the other files. The command
```sh
install(DIRECTORY data/other/ DESTINATION share/myproject
        PATTERN ".git" EXCLUDE
        PATTERN "*.txt"
          PERMISSIONS OWNER_READ OWNER_WRITE
        PATTERN "*.sh"
          PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE)
```

will install the contents of `data/other` from the source directory to `share/myproject` while ignoring .git directories and giving specific permissions to `.txt` and `.sh` files.
