# Building Your Project
After you have run CMake, your project will be ready to be built. If your target generator is based on Makefiles then you can build your project by changing the directory to your binary tree and typing make (or gmake or nmake as appropriate). If you generated files for an IDE such as Visual Studio, you can start your IDE, load the project files into it, and build as you normally would.

Another option is to use [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1))’s `--build` option from the command line. This option is simply a convenience that allows you to build your project from the command line, even if that requires launching an IDE.

That is all there is to installing and running CMake for simple projects. In the following chapters, we will consider CMake in more detail and explain how to use it on more complex software projects.
