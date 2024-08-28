# Setting Initial Values for CMake
Sometimes you may need to set up cache entries without running a GUI. This is common when setting up nightly dashboards, or if you will be creating many build trees with the same cache values. In these cases, the CMake cache can be initialized in two different ways. The first way is to pass the cache values on the CMake command line using `-DCACHE_VAR:TYPE=VALUE` arguments. For example, consider the following nightly dashboard script for a UNIX machine:
```sh
#!/bin/tcsh

cd ${HOME}

# wipe out the old binary tree and then create it again
rm -rf Foo-Linux
mkdir Foo-Linux
cd Foo-Linux

# run cmake to setup the cache
cmake -DBUILD_TESTING:BOOL=ON <etc...> ../Foo

# generate the dashboard
ctest -D Nightly
```

The same idea can be used with a batch file on Windows.

The second way is to create a file to be loaded using [`cmake`](https://cmake.org/cmake/help/latest/manual/cmake.1.html#manual:cmake(1))’s `-C` option. In this case, instead of setting up the cache with `-D` options, it is done through a file that is parsed by CMake. The syntax for this file is the standard CMakeLists syntax, which is typically a series of [`set`](https://cmake.org/cmake/help/latest/command/set.html#command:set) commands such as:
```sh
# Build the vtkHybrid kit.
set(VTK_USE_HYBRID ON CACHE BOOL "doc string")
```

In some cases there might be an existing cache, and you want to force the cache values to be set a certain way. For example, say you want to turn Hybrid on even if the user has previously run CMake and turned it off. Then you can do
```sh
# Build the vtkHybrid kit always.
set(VTK_USE_HYBRID ON CACHE BOOL "doc" FORCE)
```

Another option is that you want to set and then hide options so the user will not be tempted to adjust them later on. This can be done using type `INTERNAL`. `INTERNAL` cache variables imply FORCE and are never shown in cache editors.
```sh
# Build the vtkHybrid kit always and don't distract
# the user by showing the option.
set(VTK_USE_HYBRID ON CACHE INTERNAL "doc")
```
