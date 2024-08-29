# How to Pass Parameters to a Compilation
Once you have determined the features of the system in which you are interested, it is time to configure the software based on what has been found. There are two common ways to pass this information to the compiler: on the compile line, or using a pre-configured header. The first way is to pass definitions on the compile line. A preprocessor definition can be passed to the compiler from a CMakeLists file with the [`target_compile_definitions`](https://cmake.org/cmake/help/latest/command/target_compile_definitions.html#command:target_compile_definitions) command. For example, a common practice in C code is to have the ability to selectively compile in/out debug statements.
```sh
#ifdef DEBUG_BUILD
  printf("the value of v is %d", v);
#endif
```

A CMake variable could be used to turn on or off debug builds using the [`option`](https://cmake.org/cmake/help/latest/command/option.html#command:option) command:
```sh
option(DEBUG_BUILD
      "Build with extra debug print messages.")

if(DEBUG_BUILD)
  target_compile_definitions(mytarget PUBLIC DEBUG_BUILD)
endif()
```

Another example would be to tell the compiler the result of the previous `HAS_FOOBAR_CALL` test that was discussed earlier in this chapter. You could do this with the following:
```sh
if (HAS_FOOBAR_CALL)
  target_compile_definitions(mytarget PUBLIC HAS_FOOBAR_CALL)
endif()
```
