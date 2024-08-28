# Commands
A command consists of the command name, opening parenthesis, whitespace separated arguments, and a closing parenthesis. Each command is evaluated in the order that it appears in the CMakeLists file. See the [`cmake-commands`](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html#manual:cmake-commands(7)) manual for a full list of CMake commands.

CMake is no longer case sensitive to command names, so where you see `command`, you could use `COMMAND` or `Command` instead. It is considered best practice to use lowercase commands. All whitespace (spaces, line feeds, tabs) is ignored except to separate arguments. Therefore, commands may span multiple lines as long as the command name and the opening parenthesis are on the same line.

CMake command arguments are space separated and case sensitive. Command arguments may be either quoted or unquoted. A quoted argument starts and ends in a double quote (“) and always represents exactly one argument. Any double quotes contained inside the value must be escaped with a backslash. Consider using bracket arguments for arguments that require escaping, see the [`cmake-language`](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#manual:cmake-language(7)) manual. An unquoted argument starts in any character other than a double quote (later double quotes are literal) and is automatically expanded into zero-or-more arguments by separating on semicolons within the value. For example:
```sh
command("")          # 1 quoted argument
command("a b c")     # 1 quoted argument
command("a;b;c")     # 1 quoted argument
command("a" "b" "c") # 3 quoted arguments
command(a b c)       # 3 unquoted arguments
command(a;b;c)       # 1 unquoted argument expands to 3
```
