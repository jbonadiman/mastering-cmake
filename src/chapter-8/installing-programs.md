# Installing Programs
Projects may also install helper programs, such as shell scripts or Python scripts that are not actually compiled as targets. These may be installed with the `FILES` signature using the `PERMISSIONS` option to add execute permission. However, this case is common enough to justify a simpler interface. CMake provides the `PROGRAMS` signature for this purpose.

The `PROGRAMS` keyword is immediately followed by a list of scripts to be installed. This command is identical to the `FILES` signature, except that the default permissions additionally include `OWNER_EXECUTE`, `GROUP_EXECUTE`, and `WORLD_EXECUTE`. For example, we may install a Python utility script with the command
```sh
install(PROGRAMS my-util.py DESTINATION bin)
```

which installs `my-util.py` to the `bin` directory under the installation prefix and gives it owner, group, world read and execute permissions, plus owner write.
