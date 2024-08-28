# Conditional Statements
First we will consider the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command. In many ways, the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command in CMake is just like the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command in any other language. It evaluates its expression and uses it to execute the code in its body or optionally the code in the [`else`](https://cmake.org/cmake/help/latest/command/else.html#command:else) clause. For example:
```sh
if(FOO)
  # do something here
else()
  # do something else
endif()
```

CMake also supports [`elseif`](https://cmake.org/cmake/help/latest/command/elseif.html#command:elseif) to help sequentially test for multiple conditions. For example:
```sh
if(MSVC80)
  # do something here
elseif(MSVC90)
  # do something else
elseif(APPLE)
  # do something else
endif()
```

The [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command documents the many conditions it can test.
