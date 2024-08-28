# Silence a Warning by Updating Code
When a project does not work correctly with the NEW behaviors for a policy, the code needs to be updated. In order to deal with a warning for some policy `CMP<NNNN>`, add
```sh
cmake_policy(SET CMP<NNNN> NEW)
```

to the top of the project and then fix the code to work with the NEW behavior.

If many instances of the warning occur fixing all of them simultaneously may be too difficult: instead, a developer may fix them one at a time by using the PUSH/POP signatures of the [`cmake_policy`](https://cmake.org/cmake/help/latest/command/cmake_policy.html#command:cmake_policy) command:
```sh
cmake_policy(PUSH)
cmake_policy(SET CMP<NNNN> NEW)
# ... code updated for new policy behavior ...
cmake_policy(POP)
```

This will request the new behavior for a small region of code that has been fixed. Other instances of the policy warning may still appear and must be fixed separately.
