# CMake Package Registry
CMake provides two central locations to register packages that have been built or installed anywhere on a system: a *User Package Registry* and a *System Package Registry*. The [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command searches the two package registries as two of the search steps specified in its documentation. The registries are especially useful for helping projects find packages in non-standard install locations or directly in the package build trees. A project may populate either the user or system registry (using its own means) to refer to its location. In either case, the package should store a package configuration file at the registered location and optionally a package version file earlier in this chapter.

The *User Package Registry* is stored in a platform-specific, per-user location. On Windows it is stored in the Windows registry under a key in `HKEY_CURRENT_USER`. A `<package>` may appear under registry key
```cmake
HKEY_CURRENT_USER\Software\Kitware\CMake\Packages\<package>
```

as a `REG_SZ` value with arbitrary name that specifies the directory containing the package configuration file. On UNIX platforms, the user package registry is stored in the user home directory under `~/.cmake/packages`. A `<package>` may appear under the directory
```cmake
~/.cmake/packages/<package>
```

as a file with arbitrary name whose content specifies the directory containing the package configuration file. The [`export(PACKAGE)`](https://cmake.org/cmake/help/latest/command/export.html#command:export) command may be used to register a project build tree in the user package registry. CMake does not currently provide an interface to add install trees to the user package registry; installers must be manually taught to register their packages if desired.

The *System Package Registry* is stored in a platform-specific, system-wide location. On Windows it is stored in the Windows registry under a key in `HKEY_LOCAL_MACHINE`. A `<package>` may appear under registry key
```cmake
HKEY_LOCAL_MACHINE\Software\Kitware\CMake\Packages\<package>
```

as a `REG_SZ` value with arbitrary name that specifies the directory containing the package configuration file. There is no system package registry on non-Windows platforms. CMake does not provide an interface to add to the system package registry; installers must be manually taught to register their packages if desired.

Package registry entries are individually owned by the project installations that they reference. A package installer is responsible for adding its own entry and the corresponding uninstaller is responsible for removing it. However, in order to keep the registries clean, the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) command automatically removes stale package registry entries it encounters if it has sufficient permissions. An entry is considered stale if it refers to a directory that does not exist or does not contain a matching package configuration file. This is particularly useful for user package registry entries created by the [`export(PACKAGE)`](https://cmake.org/cmake/help/latest/command/export.html#command:export) command for build trees which have no uninstall event and are simply deleted by developers.

Package registry entries may have arbitrary name. A simple convention for naming them is to use content hashes, as they are deterministic and unlikely to collide. The [`export(PACKAGE)`](https://cmake.org/cmake/help/latest/command/export.html#command:export) command uses this approach. The name of an entry referencing a specific directory is simply the content hash of the directory path itself. For example, a project may create package registry entries such as
```cmake
> reg query HKCU\Software\Kitware\CMake\Packages\MyPackage
HKEY_CURRENT_USER\Software\Kitware\CMake\Packages\MyPackage
 45e7d55f13b87179bb12f907c8de6fc4
                          REG_SZ    c:/Users/Me/Work/lib/cmake/MyPackage
 7b4a9844f681c80ce93190d4e3185db9
                          REG_SZ    c:/Users/Me/Work/MyPackage-build
```

on Windows, or
```cmake
$ cat ~/.cmake/packages/MyPackage/7d1fb77e07ce59a81bed093bbee945bd
/home/me/work/lib/cmake/MyPackage
$ cat ~/.cmake/packages/MyPackage/f92c1db873a1937f3100706657c63e07
/home/me/work/MyPackage-build
```

on UNIX. The command [`find_package(MyPackage)`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) will search the registered locations for package configuration files. The search order among package registry entries for a single package is unspecified. Registered locations may contain package version files to tell [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package) whether a specific location is suitable for the version requested.
