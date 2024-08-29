# Installing Code
Custom installation scripts, as simple as the message above, are more easily created with the script code placed inline in the call to the [`install`](https://cmake.org/cmake/help/latest/command/install.html#command:install) command. The `CODE` signature is provided for this purpose.

The `CODE` keyword is immediately followed by a string containing the code to place in the installation script. An install-time message may be created using the command
```cmake
install(CODE "MESSAGE(\"Installing My Project\")")
```

which has the same effect as the `message.cmake` script but contains the code inline.
