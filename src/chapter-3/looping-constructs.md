# Looping Constructs
The [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) and [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) commands allow you to handle repetitive tasks that occur in sequence. The [`break`](https://cmake.org/cmake/help/latest/command/break.html#command:break) command breaks out of a [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) or [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) loop before it would normally end.

The [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) command enables you to execute a group of CMake commands repeatedly on the members of a list. Consider the following example adapted from VTK.
```sh
foreach(tfile
        TestAnisotropicDiffusion2D
        TestButterworthLowPass
        TestButterworthHighPass
        TestCityBlockDistance
        TestConvolve
        )
  add_test(${tfile}-image ${VTK_EXECUTABLE}
    ${VTK_SOURCE_DIR}/Tests/rtImageTest.tcl
    ${VTK_SOURCE_DIR}/Tests/${tfile}.tcl
    -D ${VTK_DATA_ROOT}
    -V Baseline/Imaging/${tfile}.png
    -A ${VTK_SOURCE_DIR}/Wrapping/Tcl
    )
endforeach()
```

The first argument of the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) command is the name of the variable that will take on a different value with each iteration of the loop; the remaining arguments are the list of values over which to loop. In this example, the body of the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop is just one CMake command, [`add_test`](https://cmake.org/cmake/help/latest/command/add_test.html#command:add_test). In the body of the [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach), each time the loop variable (`tfile` in this example) is referenced will be replaced with the current value from the list. In the first iteration, occurrences of `${tfile}` will be replaced with `TestAnisotropicDiffusion2D`. In the next iteration, `${tfile}` will be replaced with `TestButterworthLowPass`. The [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop will continue to loop until all of the arguments have been processed.

It is worth mentioning that [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loops can be nested, and that the loop variable is replaced prior to any other variable expansion. This means that in the body of a [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop, you can construct variable names using the loop variable. In the code below, the loop variable `tfile` is expanded, and then concatenated with `_TEST_RESULT`. The new variable name is then expanded and tested to see if it matches `FAILED`.
```sh
if(${${tfile}_TEST_RESULT} MATCHES FAILED)
  message("Test ${tfile} failed.")
endif()
```

The [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) command provides looping based on a test condition. The format for the test expression in the [`while`](https://cmake.org/cmake/help/latest/command/while.html#command:while) command is the same as it is for the [`if`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command, as described earlier. Consider the following example, which is used by CTest. Note that CTest updates the value of `CTEST_ELAPSED_TIME` internally.
```sh
#####################################################
# run paraview and ctest test dashboards for 6 hours
#
while(${CTEST_ELAPSED_TIME} LESS 36000)
  set(START_TIME ${CTEST_ELAPSED_TIME})
  ctest_run_script("dash1_ParaView_vs71continuous.cmake")
  ctest_run_script("dash1_cmake_vs71continuous.cmake")
endwhile()
```
