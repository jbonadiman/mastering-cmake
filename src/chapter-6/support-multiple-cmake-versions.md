# Supporting Multiple CMake Versions
Some projects might want to support a few releases of CMake simultaneously. The goal is to build with an older version, while also working with newer versions without warnings. In order to support both CMake 2.4 and 2.6, one may write code like
```cmake
cmake_minimum_required(VERSION 2.4)
if(COMMAND cmake_policy)
  # policy settings ...
  cmake_policy(SET CMP0003 NEW)
endif()
```

This will set the policies to build with CMake 2.6 and to ignore them for CMake 2.4. In order to support both CMake 2.6 and some policies of CMake 2.8, one may write code like:
```cmake
cmake_minimum_required(VERSION 2.6)
if(POLICY CMP1234)
  # policies not known to CMake 2.6 ...
  cmake_policy(SET CMP1234 NEW)
endif()
```

This will set the policies to build with CMake 2.8 and to ignore them for CMake 2.6. If it is known that the project builds with both CMake 2.6 and CMake 2.8’s new policies users may write:
```cmake
cmake_minimum_required(VERSION 2.6)
if (NOT ${CMAKE_VERSION} VERSION_LESS 2.8)
   cmake_policy(VERSION 2.8)
endif()
```
