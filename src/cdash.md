# CDash
As your project’s testing needs grow, keeping track of the test results can become overwhelming. This is especially true for projects that are tested nightly on a number of different platforms. In these cases, we recommend using a test dashboard to summarize the test results.

A test dashboard summarizes the results for many tests on many platforms, and its hyperlinks allow people to drill down into additional levels of detail quickly. The CTest executable includes support for producing test dashboards. When run with the correct options, CTest will produce XML-based output recording the build and test results, and post them to a dashboard server. The dashboard server runs an open source software package called CDash. CDash collects the XML results and produces HTML web pages from them.

Before discussing how to use CTest to produce a dashboard, let us consider the main parts of a testing dashboard. Each night at a specified time, the dashboard server will open up a new dashboard so each day there is a new web page showing the results of tests for that twenty-four hour period. There are links on the main page that allow you to quickly navigate through different days. Looking at the main page for a project (such as CMake’s dashboard off of <https://www.cmake.org>), you will see that it is divided into a few main components. Near the top you will find a set of links that allow you to step to previous dashboards, as well as links to project pages such as the bug tracker, documentation, etc.

Below that, you will find groups of results. Typically groups that you will find include Nightly, Experimental, Continuous, Coverage, and Dynamic Analysis. The category into which a dashboard entry will be placed depends on how it was generated. The simplest are Experimental entries which represent dashboard results for someone’s current copy of the project’s source code. With an experimental dashboard, the source code is not guaranteed to be up to date. In contrast a Nightly dashboard entry is one where CTest tries to update the source code to a specific date and time. The expectation is that all nightly dashboard entries for a given day should be based on the same source code.

A continuous dashboard entry is one that is designed to run every time new files are checked in. Depending on how frequently new files are checked in a single day’s dashboard could have many continuous entries. Continuous dashboards are particularly helpful for cross platform projects where a problem may only show up on some platforms. In those cases a developer can commit a change that works for them on their platform and then another platform running a continuous build could catch the error, allowing the developer to correct the problem promptly.

Dynamic Analysis and Coverage dashboards are designed to test the memory safety and code coverage of a project. A Dynamic Analysis dashboard entry is one where all the tests are run with a memory access/leak checking program enabled. Any resulting errors or warnings are parsed, summarized and displayed. This is important to verify that your software is not leaking memory, or reading from uninitialized memory. Coverage dashboard entries are similar in that all the tests are run, but as they are the lines of code being executed are tracked. When all the tests have been run, a listing of how many times each line of code was executed is produced and displayed on the dashboard.

## Adding CDash Dashboard Support to a Project
The easiest way to start using CDash is to register an account and create a new project at <https://my.cdash.org>.

If you’d prefer to install your own CDash server, please follow one of these guides:

- [Installation guide](https://github.com/Kitware/CDash/blob/master/docs/install.md)
- [Docker instructions](https://github.com/Kitware/CDash/blob/master/docs/docker.md)

## Client Setup
To support dashboards in your project you need to include the [CTest](https://cmake.org/cmake/help/latest/module/CTest.html#module:CTest) module as follows.
```cmake
# Include CDash dashboard testing module
include(CTest)
```
The CTest module will then read settings from the `CTestConfig.cmake` file you created or downloaded from CDash. If you have added [`add_test`](https://cmake.org/cmake/help/latest/command/add_test.html#command:add_test) command calls to your project creating a dashboard entry is as simple as running:

The `-D` option tells CTest to create a dashboard entry. The next argument indicates what type of dashboard entry to create. Creating a dashboard entry involves quite a few steps that can be run independently, or as one command. In this example, the Experimental argument will cause CTest to perform a number of different steps as one command. The different steps of creating a dashboard entry are summarized below.

- **Start**

    Prepare a new dashboard entry. This creates a `Testing` subdirectory in the build directory. The `Testing` subdirectory will contain a subdirectory for the dashboard results with a name that corresponds to the dashboard time. The `Testing` subdirectory will also contain a subdirectory for the temporary testing results called `Temporary`.

- **Update**

    Perform a source control update of the source code (typically used for nightly or continuous runs). Currently CTest supports Concurrent Versions System (CVS), Subversion, Git, Mercurial, and Bazaar.

- **Configure**

    Run CMake on the project to make sure the Makefiles or project files are up to date.

- **Build**

    Build the software using the specified generator.

- **Test**

    Run all the tests and record the results.

- **MemoryCheck**

    Perform memory checks using Purify or valgrind.

- **Coverage**

    Collect source code coverage information using gcov or Bullseye.

- **Submit**

    Submit the testing results as a dashboard entry to the server.

Each of these steps can be run independently for a Nightly or Experimental entry using the following syntax:
```shell
ctest -D NightlyStart
ctest -D NightlyBuild
ctest -D NightlyCoverage -D NightlySubmit
```

or
```shell
ctest -D ExperimentalStart
ctest -D ExperimentalConfigure
ctest -D ExperimentalCoverage -D ExperimentalSubmit
```

Alternatively, you can use shortcuts that perform the most common combinations all at once. The shortcuts that CTest has defined include:
- **ctest -D Experimental**

    performs the start, configure, build, test, coverage, and submit commands.

- **ctest -D Nightly**

    performs the start, update, configure, build, test, coverage, and submit commands.

- **ctest -D Continuous**

    performs the start, update, configure, build, test, coverage, and submit commands.

- **ctest -D MemoryCheck**

    performs the start, configure, build, memorycheck, coverage, and submit commands.

When first setting up a dashboard it is often useful to combine the `-D` option with the `-V` option. This will allow you to see the output of all the different stages of the dashboard process. Likewise, CTest maintains log files in the `Testing/Temporary` directory it creates in your binary tree. There you will find log files for the most recent dashboard run. The dashboard results (XML files) are stored in the `Testing` directory as well.

## Customizing Dashboards for a Project
CTest has a few options that can be used to control how it processes a project. If, when CTest runs a dashboard, it finds `CTestCustom.ctest` files in the binary tree, it will load these files and use the settings from them to control its behavior. The syntax of a CTestCustom file is the same as regular CMake syntax. That said, only set commands are normally used in this file. These commands specify properties that CTest will consider when performing the testing.

### Dashboard Submissions Settings
A number of the basic dashboard settings are provided in the file that you download from CDash. You can edit these initial values and provide additional values if you wish. The first value that is set is the nightly start time. This is the time that dashboards all around the world will use for checking out their copy of the nightly source code. This time also controls how dashboard submissions will be grouped together. All submissions from the nightly start time until the next nightly start time will be included on the same “day”.
```cmake
# Dashboard is opened for submissions for a 24 hour period
# starting at the specified NIGHTLY_START_TIME. Time is
# specified in 24 hour format.
set (CTEST_NIGHTLY_START_TIME "01:00:00 UTC")
```

The next group of settings control where to submit the testing results. This is the location of the CDash server.
```cmake
# CDash server to submit results (used by client)
set (CTEST_DROP_METHOD http)
set (CTEST_DROP_SITE "my.cdash.org")
set (CTEST_DROP_LOCATION "/submit.php?project=KensTest")
set (CTEST_DROP_SITE_CDASH TRUE)
```

The [`CTEST_DROP_SITE`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_SITE.html#variable:CTEST_DROP_SITE) specifies the location of the CDash server. Build and test results generated by CDash clients are sent to this location. The [`CTEST_DROP_LOCATION`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_LOCATION.html#variable:CTEST_DROP_LOCATION) is the directory or the HTTP URL on the server where CDash clients leave their build and test reports. The [`CTEST_DROP_SITE_CDASH`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_SITE_CDASH.html#variable:CTEST_DROP_SITE_CDASH) specifies that the current server is CDash, which prevents CTest from trying to “trigger” the submission (this is still done if this variable is not set to allow for backwards compatibility with Dart and Dart 2).

Currently CDash supports only the HTTP drop submission method; however CTest supports other submission types. The [`CTEST_DROP_METHOD`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_METHOD.html#variable:CTEST_DROP_METHOD) specifies the method used to submit testing results. The most common setting for this will be HTTP which uses the Hyper Text Transfer Protocol (HTTP) to transfer the test data to the server. Other drop methods are supported for special cases such as FTP and SCP. In the example below, clients that are submitting their results using the HTTP protocol use a web address as their drop site. If the submission is via FTP, this location is relative to where the [`CTEST_DROP_SITE_USER`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_SITE_USER.html#variable:CTEST_DROP_SITE_USER) will log in by default. The [`CTEST_DROP_SITE_USER`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_SITE_USER.html#variable:CTEST_DROP_SITE_USER) specifies the FTP username the client will use on the server. For FTP submissions this user will typically be “anonymous”. However, any username that can communicate with the server can be used. For FTP servers that require a password, it can be stored in the [`CTEST_DROP_SITE_PASSWORD`](https://cmake.org/cmake/help/latest/variable/CTEST_DROP_SITE_PASSWORD.html#variable:CTEST_DROP_SITE_PASSWORD) variable. The `CTEST_DROP_SITE_MODE` (not used in this example) is an optional variable that you can use to specify the FTP mode. Most FTP servers will handle the default passive mode, but you can set the mode explicitly to active if your server does not.

CTest can also be run from behind a firewall. If the firewall allows FTP or HTTP traffic, then no additional settings are required. If the firewall requires an FTP/HTTP proxy or uses a SOCKS4 or SOCKS5 type proxy, some environment variables need to be set. HTTP_PROXY and FTP_PROXY specify the servers that service HTTP and FTP proxy requests. HTTP_PROXY_PORT and FTP_PROXY_PORT specify the port on which the HTTP and FTP proxies reside. HTTP_PROXY_TYPE specifies the type of the HTTP proxy used. The three different types of proxies supported are the default, which includes a generic HTTP/FTP proxy, “SOCKS4”, and “SOCKS5”, which specify SOCKS4 and SOCKS5 compatible proxies.

### Filtering Errors and Warnings
By default, CTest has a list of regular expressions that it matches for finding the errors and warnings from the output of the build process. You can override these settings in your `CTestCustom.ctest` files using several variables as shown below.
```cmake
set (CTEST_CUSTOM_WARNING_MATCH
  ${CTEST_CUSTOM_WARNING_MATCH}
  "{standard input}:[0-9][0-9]*: Warning: "
  )

set (CTEST_CUSTOM_WARNING_EXCEPTION
  ${CTEST_CUSTOM_WARNING_EXCEPTION}
  "tk8.4.5/[^/]+/[^/]+.c[:\"]"
  "xtree.[0-9]+. : warning C4702: unreachable code"
  "warning LNK4221"
  "variable .var_args[2]*. is used before its value is set"
  "jobserver unavailable"
  )
```

Another useful feature of the CTestCustom files is that you can use it to limit the tests that are run for memory checking dashboards. Memory checking using purify or valgrind is a CPU intensive process that can take twenty hours for a dashboard that normally takes one hour. To help alleviate this problem, CTest allows you to exclude some of the tests from the memory checking process as follows:
```cmake
set (CTEST_CUSTOM_MEMCHECK_IGNORE
     ${CTEST_CUSTOM_MEMCHECK_IGNORE}
  TestSetGet
  otherPrint-ParaView
  Example-vtkLocal
  Example-vtkMy
  )
```

The format for excluding tests is simply a list of test names as specified when the tests were added in your CMakeLists file with [`add_test`](https://cmake.org/cmake/help/latest/command/add_test.html#command:add_test).

In addition to the demonstrated settings, such as `CTEST_CUSTOM_WARNING_MATCH`, `CTEST_CUSTOM_WARNING_EXCEPTION`, and `CTEST_CUSTOM_MEMCHECK_IGNORE`, CTest also checks several other variables.
- **CTEST_CUSTOM_ERROR_MATCH**

    Additional regular expressions to consider a build line as an error line

- **CTEST_CUSTOM_ERROR_EXCEPTION**

    Additional regular expressions to consider a build line not as an error line

- **CTEST_CUSTOM_WARNING_MATCH**

    Additional regular expressions to consider a build line as a warning line

- **CTEST_CUSTOM_WARNING_EXCEPTION**

    Additional regular expressions to consider a build line not as a warning line

- **CTEST_CUSTOM_MAXIMUM_NUMBER_OF_ERRORS**

    Maximum number of errors before CTest stops reporting errors (default 50)

- **CTEST_CUSTOM_MAXIMUM_NUMBER_OF_WARNINGS**

    Maximum number of warnings before CTest stops reporting warnings (default 50)

- **CTEST_CUSTOM_COVERAGE_EXCLUDE**

    Regular expressions for files to be excluded from the coverage analysis

- **CTEST_CUSTOM_PRE_MEMCHECK**

    List of commands to execute before performing memory checking

- **CTEST_CUSTOM_POST_MEMCHECK**

    List of commands to execute after performing memory checking

- **CTEST_CUSTOM_MEMCHECK_IGNORE**

    List of tests to exclude from the memory checking step

- **CTEST_CUSTOM_PRE_TEST**

    List of commands to execute before performing testing

- **CTEST_CUSTOM_POST_TEST**

    List of commands to execute after performing testing

- **CTEST_CUSTOM_TESTS_IGNORE**

    List of tests to exclude from the testing step

- **CTEST_CUSTOM_MAXIMUM_PASSED_TEST_OUTPUT_SIZE**

    Maximum size of test output for the passed test (default 1k)

- **CTEST_CUSTOM_MAXIMUM_FAILED_TEST_OUTPUT_SIZE**

    Maximum size of test output for the failed test (default 300k)

Commands specified in `CTEST_CUSTOM_PRE_TEST` and `CTEST_CUSTOM_POST_TEST`, as well as the equivalent memory checking ones, are executed once per CTest run. These commands can be used, for example, if all tests require some initial setup and some final cleanup to be performed.

### Adding Notes to a Dashboard
CTest and CDash support adding note files to a dashboard submission. These will appear on the dashboard as a clickable icon that links to the text of all the files. To add notes, call CTest with the -A option followed by a semicolon-separated list of filenames. The contents of these files will be submitted as notes for the dashboard. For example:
```shell
ctest -D Continuous -A C:/MyNotes.txt;C:/OtherNotes.txt
```

Another way to submit notes with a dashboard is to copy or write the notes as files into a Notes directory under the `Testing` directory of your binary tree. Any files found there when CTest submits a dashboard will also be uploaded as notes.

## CTest Scripting
This section describes how to write command-based CTest scripts that allow the maintainer to have fine-grained control over the individual steps of a dashboard.

The dashboard maintainer has access to individual CTest command functions, such as [`ctest_configure`](https://cmake.org/cmake/help/latest/command/ctest_configure.html#command:ctest_configure) and [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build). By running these functions individually, the user can flexibly develop custom testing schemes. Here’s an example of a CTest script
```cmake
cmake_minimum_required(VERSION 3.20)

set(CTEST_SITE          "andoria.kitware")
set(CTEST_BUILD_NAME    "Linux-g++")
set(CTEST_NOTES_FILES
    "${CTEST_SCRIPT_DIRECTORY}/${CTEST_SCRIPT_NAME}")

set(CTEST_DASHBOARD_ROOT   "$ENV{HOME}/Dashboards/My Tests")
set(CTEST_SOURCE_DIRECTORY "${CTEST_DASHBOARD_ROOT}/CMake")
set(CTEST_BINARY_DIRECTORY "${CTEST_DASHBOARD_ROOT}/CMake-gcc ")

set(CTEST_UPDATE_COMMAND    "/usr/bin/cvs")
set(CTEST_CONFIGURE_COMMAND
    "\"${CTEST_SOURCE_DIRECTORY}/bootstrap\"")
set(CTEST_BUILD_COMMAND     "/usr/bin/make -j 2")


ctest_empty_binary_directory(${CTEST_BINARY_DIRECTORY})

ctest_start(Nightly)
ctest_update(SOURCE "${CTEST_SOURCE_DIRECTORY}")
ctest_configure(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_build(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_test(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_submit()
```

The first block contains the variables about the submission.
```cmake
set(CTEST_SITE              "andoria.kitware")
set(CTEST_BUILD_NAME        "Linux-g++")
set(CTEST_NOTES_FILES
    "${CTEST_SCRIPT_DIRECTORY}/${CTEST_SCRIPT_NAME}")
```

These variables are used to identify the system once it submits the results to the dashboard. `CTEST_NOTES_FILES` is a list of files that should be submitted as the notes of the dashboard submission. This variable corresponds to the -A flag of [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1)).

The second block describes the information that CTest functions will use to perform the tasks:
```cmake
set(CTEST_DASHBOARD_ROOT   "$ENV{HOME}/Dashboards/My Tests")
set(CTEST_SOURCE_DIRECTORY "${CTEST_DASHBOARD_ROOT}/CMake")
set(CTEST_BINARY_DIRECTORY "${CTEST_DASHBOARD_ROOT}/CMake-gcc ")
set(CTEST_UPDATE_COMMAND    "/usr/bin/cvs")
set(CTEST_CONFIGURE_COMMAND
   "\"${CTEST_SOURCE_DIRECTORY}/bootstrap\"")
set(CTEST_BUILD_COMMAND     "/usr/bin/make -j 2")
```

The `CTEST_UPDATE_COMMAND` is the path to the command used to update the source directory from the repository. Currently CTest supports Concurrent Versions System (CVS), Subversion, Git, Mercurial, and Bazaar.

Both the configure and build handlers support two modes. One mode is to provide the full command that will be invoked during that stage. This is designed to support projects that do not use CMake as their configuration or build tool. In this case, you specify the full command lines to configure and build your project by setting the [`CTEST_CONFIGURE_COMMAND`](https://cmake.org/cmake/help/latest/variable/CTEST_CONFIGURE_COMMAND.html#variable:CTEST_CONFIGURE_COMMAND) and [`CTEST_BUILD_COMMAND`](https://cmake.org/cmake/help/latest/variable/CTEST_BUILD_COMMAND.html#variable:CTEST_BUILD_COMMAND) variables respectively.

For the build step you should also set the variables `CTEST_PROJECT_NAME` and `CTEST_BUILD_CONFIGURATION`, to specify how to build the project. In this case `CTEST_PROJECT_NAME` will match the top level CMakeLists file’s [`project`](https://cmake.org/cmake/help/latest/command/project.html#command:project) command. The `CTEST_BUILD_CONFIGURATION` should be one of Release, Debug, MinSizeRel, or RelWithDebInfo. Additionally, `CTEST_BUILD_FLAGS` can be provided as a hint to the build command. An example of testing for a CMake based project would be:
```cmake
set(CTEST_PROJECT_NAME "Grommit")
set(CTEST_BUILD_CONFIGURATION "Debug")
```

The final block performs the actual testing and submission:
```cmake
ctest_empty_binary_directory(${CTEST_BINARY_DIRECTORY})

ctest_start(Nightly)

ctest_update(SOURCE
             "${CTEST_SOURCE_DIRECTORY}" RETURN_VALUE res)
ctest_configure(BUILD
                "${CTEST_BINARY_DIRECTORY}" RETURN_VALUE res)
ctest_build(BUILD "${CTEST_BINARY_DIRECTORY}" RETURN_VALUE res)
ctest_test(BUILD "${CTEST_BINARY_DIRECTORY}" RETURN_VALUE res)
ctest_submit(RETURN_VALUE res)
```

The [`ctest_empty_binary_directory`](https://cmake.org/cmake/help/latest/command/ctest_empty_binary_directory.html#command:ctest_empty_binary_directory) command empties the directory and all subdirectories. Please note that this command has a safety measure built in, which is that it will only remove the directory if there is a CMakeCache.txt file in the top level directory. This was intended to prevent CTest from mistakenly removing a non-build directory.

The rest of the block contains the calls to the actual CTest functions. Each of them corresponds to a CTest -D option. For example, instead of:
```shell
ctest -D ExperimentalBuild
```

the script would contain:
```cmake
ctest_start(Experimental)
ctest_build(BUILD "${CTEST_BINARY_DIRECTORY}" RETURN_VALUE res)
```

Each step yields a return value, which indicates if the step was successful. For example, the return value of the Update stage can be used in a continuous dashboard to determine if the rest of the dashboard should be run.

Let us examine a more advanced CTest script. This script drives testing of an application called Slicer. Slicer uses CMake internally, but it drives the build process through a series of Tcl scripts. One of the problems of this approach is that it does not support out-of-source builds. Also, on Windows certain modules come pre-built, so they have to be copied to the build directory. To test a project like that, we would use a script like this:
```cmake
cmake_minimum_required(VERSION 3.20)

# set the dashboard specific variables -- name and notes
set(CTEST_SITE              "dash11.kitware")
set(CTEST_BUILD_NAME        "Win32-VS71")
set(CTEST_NOTES_FILES
     "${CTEST_SCRIPT_DIRECTORY}/${CTEST_SCRIPT_NAME}")

# do not let any single test run for more than 1500 seconds
set(CTEST_TIMEOUT "1500")

# set the source and binary directories
set(CTEST_SOURCE_DIRECTORY  "C:/Dashboards/MyTests/slicer2")
set(CTEST_BINARY_DIRECTORY  "${CTEST_SOURCE_DIRECTORY}-build")

set (SLICER_SUPPORT
     "//Dash11/Shared/Support/SlicerSupport/Lib")
set (TCLSH   "${SLICER_SUPPORT}/win32/bin/tclsh84.exe")
# set the complete update, configure and build commands
set (CTEST_UPDATE_COMMAND
     "C:/Program Files/TortoiseCVS/cvs.exe")
set (CTEST_CONFIGURE_COMMAND
    "\"${TCLSH}\"
     \"${CTEST_BINARY_DIRECTORY}/Scripts/genlib.tcl\"")
set (CTEST_BUILD_COMMAND
    "\"${TCLSH}\"
     \"${CTEST_BINARY_DIRECTORY}/Scripts/cmaker.tcl\"")

# clear out the binary tree
file (WRITE "${CTEST_BINARY_DIRECTORY}/CMakeCache.txt"
      "// Dummy cache just so that ctest will wipe binary dir")
ctest_empty_binary_directory (${CTEST_BINARY_DIRECTORY})

# special variables for the Slicer build process
set (ENV{MSVC6}              "0")
set (ENV{GENERATOR}          "Visual Studio 7 .NET 2003")
set (ENV{MAKE}               "devenv.exe ")
set (ENV{COMPILER_PATH}
     "C:/Program Files/Microsoft Visual Studio .NET
2003/Common7/Vc7/bin")
set (ENV{CVS}                "${CTEST_UPDATE_COMMAND}")

# start and update the dashboard
ctest_start (Nightly)
ctest_update (SOURCE "${CTEST_SOURCE_DIRECTORY}")

# define a macro to copy a directory
macro (COPY_DIR srcdir destdir)
  exec_program ("${CMAKE_EXECUTABLE_NAME}" ARGS
               "-E copy_directory \"${srcdir}\" \"${destdir}\"")
endmacro ()

# Slicer does not support out of source builds so we
# first copy the source directory to the binary directory
# and then build it
copy_dir ("${CTEST_SOURCE_DIRECTORY}"
          "${CTEST_BINARY_DIRECTORY}")


# copy support libraries that slicer needs into the binary tree
copy_dir ("${SLICER_SUPPORT}"
          "${CTEST_BINARY_DIRECTORY}/Lib")

# finally do the configure, build, test and submit steps
ctest_configure (BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_build (BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_test (BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_submit ()
```

With extended CTest scripting we have full control over the flow, so we can perform arbitrary commands at any point. For example, after performing an update of the project, the script copies the source tree into the build directory. This allows it to do an “out-of-source” build.

### Project Roles: CDash supports three role levels for users
- Normal users are regular users with read and/or write access to the project’s code repository.
- Site maintainers are responsible for periodic submissions to CDash.
- Project administrators have reserved privileges to administer the project in CDash.

The first two levels can be defined by the users themselves. Project administrator access must be granted by another administrator of the project, or a CDash server administrator.

### Submission backup
CDash backups all the incoming XML submissions and places them in the `backup` directory by default. The default timeframe is 48 hours. The timeframe can be changed in the `config.local.php` as follows:
```php
$CDASH_BACKUP_TIMEFRAME=72;
```

If projects are private it is recommended to set the backup directory outside of the apache root directory to make sure that nobody can access the XML files, or to add the following lines to the .htaccess in the backup directory:
```xml
<Files *>
order allow,deny
deny from all
</Files>
```

Note that the backup directory is emptied only when a new submission arrives. If necessary, CDash can also import builds from the backup directory.

### Build Groups
Builds can be organized by groups. In CDash, three groups are defined automatically and cannot be removed: `Nightly`, `Continuous` and `Experimental`. These groups are the same as the ones imposed by CTest. Each group has an associated description that is displayed when clicking on the name of the group on the main dashboard.

By default, a build belongs to the group associated with the build type defined by CTest, i.e. a nightly build will go in the Nightly section. CDash matches a build by its name, site, and build type. For instance, a nightly build named “Linux-gcc-4.3” from the site “midworld.kitware will be moved to the Nightly section unless a rule on “Linux-gcc-4.3”-“midworld.kitware”-“Nightly” is defined. There are two ways to move a build into a given group by defining a rule: Global Move and Single Move.

#### Single move allows modifying only a particular build
If logged in as an administrator of the project, a small folder icon is displayed next to each build on the main dashboard page. Clicking on the icon shows some options for each build. In particular, project administrators can mark a build as expected, move a build to a specific group, or delete a bogus build.

Expected builds: Project administrators can mark certain builds as expected. That means builds are expected to submit daily. This allows you to quickly check if a build has not been submitting on today’s dashboard, or to quickly assess how long the build has been missing by clicking on the info icon on the main dashboard.

If an expected build was not submitted the previous day and the option “Email Build Missing” is checked for the project, an email will be sent to the site maintainer and project administrator to alert them (see the Sites section for more information).

### Email
CDash sends email to developers and project administrators when a failure occurs for a given build. The configuration of the email feature is located in three places: the `config.local.php` file, the project administration section, and the project’s groups section.

In the `config.local.php`, two variables are defined to specify the email address from which email is sent and the reply address. Note that the SMTP server cannot be defined in the current version of CDash, it is assumed that a local email server is running on the machine.
```php
$CDASH_EMAIL_FROM = 'admin@mywebsite.com';
$CDASH_EMAIL_REPLY = 'noreply@mywebsite.com';
```

In the email configuration section of the project, several parameters can be tuned to control the email feature.

In the “build groups” administration section of a project, an administrator can decide if emails are sent to a specific group, or if only a summary email should be sent. The summary email is sent for a given group when at least one build is failing on the current day.

### Sites
CDash refers to a site as an individual machine submitting at least one build to a given project. A site might submit multiple builds (e.g. nightly and continuous) to multiple projects stored in CDash.

In order to see the site description, click on the name of the site from the main dashboard page for a project. The description of a site includes information regarding the processor type and speed, as well as the amount of memory available on the given machine. The description of a site is automatically sent by CTest, however in some cases it might be required to manually edit it. Moreover, if the machine is upgraded, i.e. the memory is upgraded; CDash keeps track of the history of the description, allowing users to compare performance before and after the upgrade.

Sites usually belong to one maintainer, responsible for the submissions to CDash. It is important for site maintainers to be warned when a site is not submitting as it could be related to a configuration issue.

Once a site is claimed, its maintainer will receive emails if the client machine does not submit for an unknown reason, assuming that the site is expected to submit nightly. Furthermore, the site will appear in the “My Sites” section of the maintainer’s profile, facilitating a quick check of the site’s status.

Another feature of the site page is the pie chart showing the load of the machine. Assuming that a site submits to multiple projects, it is usually useful to know if the machine has room for other submissions to CDash. The pie chart gives an overview of the machine submission time for each project.

### Graphs
CDash currently plots three types of graphs. The graphs are generated dynamically from the database records, and are interactive.

The build time graph displays the time required to build a project over time.

The test time graphs display the time to run a specific test as well as its status (passed/failed) over time.

### Adding Notes to a Build
In some cases, it is useful to inform other developers that someone is currently looking at the errors for a build.

### Logging
CDash supports an internal logging mechanism using the error_log() PHP function. Any critical SQL errors are logged. By default, the CDash log file is located in the backup directory under the name `cdash.log`. The location of the log file can be modified by changing the variable in the `config.local.php` configuration file.
```php
$CDASH_BACKUP_DIRECTORY='/var/temp/cdashbackup/log';
```

The log file can be accessed directly from CDash if the log file is in the standard location.

A log rotation mechanism which allows users to cap the current log file to a certain size is available.

### Test Timing
CDash supports checks on the duration of tests. CDash keeps the current weighted average of the mean and standard deviation for the time each test takes to run in the database. In order to keep the computation as efficient as possible the following formula is used, which only involves the previous build.
```
// alpha is the current "window" for the computation
// By default, alpha is 0.3
newMean = (1-alpha)*oldMean + alpha*currentTime

newSD = sqrt((1-alpha)*SD*SD +
  alpha*(currentTime-newMean)*(currentTime-newMean))
```

A test is defined as having failed timing based on the following logic:
```
if previousSD < thresholdSD then previousSD = thresholdSD
if currentTime > previousMean+multiplier*previousSD then fail
```

### Mobile Support
Since CDash is written using template layers via XSLT, developing new layouts is as simple as adding new rendering templates. As a demonstration, an iPhone web template is provided with the current version of CDash.
```
http://mycdashserver/CDash/iphone
```

The main page shows a list of the public projects hosted on the server. Clicking on the name of a project loads its current dashboard. In the same manner, clicking on a given build displays more detailed information about that build. As of this writing, the ability to login and to access private sections of CDash are not supported with this layout.

### Backing up CDash
All of the data (except the logs) used by CDash is stored in its database. It is important to backup the database regularly, especially so before performing a CDash upgrade. There are a couple of ways to backup a MySQL database. The easiest is to use the [mysqldump](http://dev.mysql.com/doc/refman/5.1/en/mysqldump.html) command:
```shell
mysqldump -r cdashbackup.sql cdash
```

If you are using MyISAM tables exclusively, you can copy the CDash directory in your MySQL data directory. Note that you need to shutdown MySQL before doing the copy so that no file could be changed during the copy. Similarly to MySQL, PostGreSQL has a pg_dump utility:
```shell
pg_dump -U posgreSQL_user cdash > cdashbackup.sql
```

### Upgrading CDash
When a new version of CDash is released or if you decide to update from the repository, CDash will warn you on the front page if the current database needs to be upgraded. When upgrading to a new release version the following steps should be taken:
1. Backup your SQL database (see previous section).
2. Backup your `config.local.php` (or `config.php`) configuration files.
3. Replace your current cdash directory with the latest version and copy the `config.local.php` in the cdash directory.
4. Navigate your browser to your CDash page. (e.g. <http://localhost/CDash>).
5. Note the version number on the main page, it should match the version that you are upgrading to.
6. The following message may appear: “The current database schema doesn’t match the version of CDash you are running, upgrade your database structure in the Administration panel of CDash.” This is a helpful reminder to perform the following steps.
7. Login to CDash as administrator.
8. In the ‘Administration’ section, click on ‘[CDash Maintenance]’.
9. Click on ‘Upgrade CDash’: this process might take some time depending on the size of your database (do not close your browser).
    - Progress messages may appear while CDash performs the upgrade.
    - If the upgrade process takes too long you can check in the `backup/cdash.log` file to see where the process is taking a long time and/or failing.
    - It has been reported that on some systems the spinning icon never turns into a check mark. Please check the `cdash.log` for the “Upgrade done.” string if you feel that the upgrade is taking too long.
    - On a 50GB database the upgrade might take up to 2 hours.
10. Some web browsers might have issues when upgrading (with some javascript variables not being passed correctly), in that case you can perform individual updates. For example, upgrading from CDash 1-2 to 1-4:
```
http://mywebsite.com/CDash/backwardCompatibilityTools.php?upgrade-1-4=1
```

### CDash Maintenance
Database maintenance: we recommend that you perform database optimization (reindexing, purging, etc.) regularly to maintain a stable database. MySQL has a utility called `mysqlcheck`, and PostgreSQL has several utilities such as `vacuumdb`.

Deleting builds with incorrect dates: some builds might be submitted to CDash with the wrong date, either because the date in the XML file is incorrect or the timezone was not recognized by CDash (mainly by PHP). These builds will not show up in any dashboard because the start time is bogus. In order to remove these builds:
1. Login to CDash as administrator.
2. Click on [CDash maintenance] in the administration section.
3. Click on ‘Delete builds with wrong start date’.

Recompute test timing: if you just upgraded CDash you might notice that the current submissions are showing a high number of failing test due to time defects. This is because CDash does not have enough sample points to compute the mean and standard deviation for each test, in particular the standard deviation might be very small (probably zero for the first few samples). You should turn the “enable test timing” off for about a week, or until you get enough build submissions and CDash has calculated an approximate mean and standard deviation for each test time.

The other option is to force CDash to compute the mean and standard deviation for each test for the past few days. Be warned that this process may take a long time, depending on the number of test and projects involved. In order to recompute the test timing:

1. Login to CDash as administrator.
2. Click on [CDash maintenance] in the administration section.
3. Specify the number of days (default is 4) to recompute the test timings for.
4. Click on “Compute test timing”. When the process is done the new mean, standard deviation, and status should be updated for the tests submitted during this period.

#### Automatic build removal
In order to keep the database at a reasonable size, CDash can automatically purge old builds. There are currently two ways to setup automatic removal of builds: without a cronjob, edit the `config.local.php` and add/edit the following line
```php
$CDASH_AUTOREMOVE_BUILDS='1';
```

CDash will automatically remove builds on the first submission of the day. Note that removing builds might add an extra load on the database, or slow down the current submission process if your database is large and the number of submissions is high. If you can use a cronjob the PHP command line tool can be used to trigger build removals at a convenient time. For example, removing the builds for all the projects at 6am every Sunday:
```shell
0 6 * * 0 php5 /var/www/CDash/autoRemoveBuilds.php all
```

#### CDash XML Schema
The XML parsers in CDash can be easily extended to support new features. The current XML schemas generated by CTest, and their features as described in the book, are located at:
```
http://public.kitware.com/Wiki/CDash:XML
```

### Subprojects
CDash supports splitting projects into subprojects. Some of the subprojects may in turn depend on other subprojects. A typical real life project consists of libraries, executables, test suites, documentation, web pages, and installers. Organizing your project into well-defined subprojects and presenting the results of nightly builds on a CDash dashboard can help identify where the problems are at different levels of granularity.

A project with subprojects has a different view for its top level CDash page than a project without any. It contains a summary row for the project as a whole, and then one summary row for each subproject.

#### Organizing and defining subprojects
To add subproject organization to your project, you must: (1) define the subprojects for CDash, so that it knows how to display them properly and (2) use build scripts with CTest to submit subproject builds of your project. Some (re-)organization of your project’s CMakeLists.txt files may also be necessary to allow building of your project by subprojects.

There are two ways to define subprojects and their dependencies: interactively in the CDash GUI when logged in as a project administrator, or by submitting a `Project.xml` file describing the subprojects and dependencies.

#### Adding Subprojects Interactively
As a project administrator, a “Manage subprojects” button will appear for each of your projects on the My CDash page. Clicking the Manage Subprojects button opens the manage subproject page, where you may add new subprojects or establish dependencies between existing subprojects for any project that you are an administrator of.

#### Adding Subprojects Automatically
Another way to define CDash subprojects and their dependencies is to submit a “Project.xml” file along with the usual submission files that CTest sends when it submits a build to CDash. To define the same two subprojects as in the interactive example above (Exes and Libs) with the same dependency (Exes depend on Libs), the `Project.xml` file would look like the following example:
```xml
<Project name="Tutorial">
  <SubProject name="Libs"></SubProject>
  <SubProject name="Exes">
    <Dependency name="Libs">
  </SubProject>
</Project>
```

Once the `Project.xml` file is written or generated, it can be submitted to CDash from a ctest -S script using the FILES argument to the [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) command, or directly from the [`ctest`](https://cmake.org/cmake/help/latest/manual/ctest.1.html#manual:ctest(1)) command line in a build tree configured for dashboard submission.

From inside a ctest -S script:
```
ctest_submit(FILES "${CTEST_BINARY_DIRECTORY}/Project.xml")
```

From the command line:
```shell
cd ../Project-build
ctest --extra-submit Project.xml
```

CDash will automatically add subprojects and dependencies according to the `Project.xml` file. CDash will also remove any subprojects or dependencies not defined in the `Project.xml` file. Additionally, if the same `Project.xml` is submitted multiple times, the second and subsequent submissions will have no observable effect: the first submission adds/modifies the data, the second and later submissions send the same data, so no changes are necessary. CDash tracks changes to the subproject definitions over time to allow for projects to evolve. If you view dashboards from a past date, CDash will present the project/subproject views according to the subproject definitions in effect on that date.

### Using ctest_submit with PARTS and FILES
The [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) command supports `PARTS` and `FILES` arguments. With `PARTS`, you can send any subset of the xml files with each [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) call. By default, all available parts are sent with any call to [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit). A script can wait until all dashboard stages are complete and then call [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) once to send the results of all stages at the end of the run. Alternatively, a script may call [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) with `PARTS` to do partial submissions of subsets of the results. For example, you can submit configure results after [`ctest_configure`](https://cmake.org/cmake/help/latest/command/ctest_configure.html#command:ctest_configure), build results after [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build), and test results after [`ctest_test`](https://cmake.org/cmake/help/latest/command/ctest_test.html#command:ctest_test). This allows for information to be posted as the builds progress.

With `FILES`, you can send arbitrary XML files to CDash. In addition to the standard build result XML files that CTest sends, CDash also handles a `Project.xml` file that describes subprojects and dependencies. Below is an example of a dashboard script that contains a single [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) call on its last line
```cmake
ctest_start(Experimental)
ctest_update(SOURCE "${CTEST_SOURCE_DIRECTORY}")
ctest_configure(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_build(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_test(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_submit()
```

Submissions can occur incrementally, with each part of the submission sent piecemeal as it becomes available:
```cmake
ctest_start(Experimental)
ctest_update(SOURCE "${CTEST_SOURCE_DIRECTORY}")
ctest_configure(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_submit(PARTS Update Configure Notes)

ctest_build(BUILD "${CTEST_BINARY_DIRECTORY}" APPEND)
ctest_submit(PARTS Build)

ctest_test(BUILD "${CTEST_BINARY_DIRECTORY}")
ctest_submit(PARTS Test)
```

Submitting incrementally by parts means that you can inspect the results of the configure stage live on the CDash dashboard while the build is still in progress. Likewise, you can inspect the results of the build stage live while the tests are still running. When submitting by parts, it’s important to use the `APPEND` keyword in the [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) command. If you don’t use `APPEND`, then CDash will erase any existing build with the same build name, site name, and build stamp when it receives the Build.xml file.

### Splitting Your Project into Multiple Subprojects
One [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) invocation that builds everything, followed by one [`ctest_test`](https://cmake.org/cmake/help/latest/command/ctest_test.html#command:ctest_test) invocation that tests everything is sufficient for a project that has no subprojects, but if you want to submit results on a per-subproject basis to CDash, you will have to make some changes to your project and test scripts. For your project you need to identify what targets are part of what subprojects. If you organize your CMakeLists files such that you have a target to build for each subproject, and you can derive (or look up) the name of that target based on the subproject name, then revising your script to separate it into multiple smaller configure/build/test chunks should be relatively painless. To do this, you can modify your CMakeLists files in various ways depending on your needs. The most common changes are listed below.

#### CMakeLists.txt modifications
- Name targets the same as subprojects, base target names on subproject names, or provide a look up mechanism to map from subproject name to target name.
- Possibly add custom targets to aggregate existing targets into subprojects, using [`add_dependencies`](https://cmake.org/cmake/help/latest/command/add_dependencies.html#command:add_dependencies) to say which existing targets the custom target depends on.
- Add the `LABELS` target property to targets with a value of the subproject name.
- Add the `LABELS` test property to tests with a value of the subproject name.

Next, you need to modify your CTest scripts that run your dashboards. To split your one large monolithic build into smaller subproject builds, you can use a [`foreach`](https://cmake.org/cmake/help/latest/command/foreach.html#command:foreach) loop in your CTest driver script. To help you iterate over your subprojects, CDash provides a variable named `CTEST_PROJECT_SUBPROJECTS` in `CTestConfig.cmake`. Given the above example, CDash produces a variable like this:
```cmake
set(CTEST_PROJECT_SUBPROJECTS Libs Exes)
```

CDash orders the elements in this list such that the independent subprojects (that do not depend on any other subprojects) are first, followed by subprojects that depend only on the independent subprojects, and after that subprojects that depend on those. The same logic continues until all subprojects are listed exactly once in this list in an order that makes sense for building them sequentially, one after the other.

To facilitate building just the targets associated with a subproject, use the variable named `CTEST_BUILD_TARGET` to tell [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) what to build. To facilitate running just the tests associated with a subproject, assign the `LABELS` test property to your tests and use the `INCLUDE_LABEL` argument to [`ctest_test`](https://cmake.org/cmake/help/latest/command/ctest_test.html#command:ctest_test).

#### ctest driver script modifications
- Iterate over the subprojects in dependency order (from independent to most dependent…).
- Set the SubProject and Label global properties – CTest uses these properties to submit the results to the correct subproject on the CDash server.
- Build the target(s) for this subproject: compute the name of the target to build from the subproject name, set `CTEST_BUILD_TARGET`, call [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build).
- Run the tests for this subproject using the `INCLUDE` or `INCLUDE_LABEL` arguments to [`ctest_test`](https://cmake.org/cmake/help/latest/command/ctest_test.html#command:ctest_test).
- Use [`ctest_submit`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) with the PARTS argument to submit partial results as they complete.

To illustrate this, the following example shows the changes required to split a build into smaller pieces. Assume that the subproject name is the same as the target name required to build the subproject’s components. For example, here is a snippet from CMakeLists.txt, in the hypothetical Tutorial project. The only additions necessary (since the target names are the same as the subproject names) are the calls to [`set_property`](https://cmake.org/cmake/help/latest/command/set_property.html#command:set_property) for each target and each test.
```cmake
# "Libs" is the library name (therefore a target name) and
# the subproject name
add_library (Libs ...)
set_property (TARGET Libs PROPERTY LABELS Libs)
add_test (LibsTest1 ...)
add_test (LibsTest2 ...)
set_property (TEST LibsTest1 LibsTest2 PROPERTY LABELS Libs)

# "Exes" is the executable name (therefore a target name)
# and the subproject name
add_executable (Exes ...)
target_link_libraries (Exes Libs)
set_property (TARGET Exes PROPERTY LABELS Exes)
add_test (ExesTest1 ...)
add_test (ExesTest2 ...)
set_property (TEST ExesTest1 ExesTest2 PROPERTY LABELS Exes)
```

Here is an example of what the CTest driver script might look like before and after organizing this project into subprojects. Before the changes
```cmake
ctest_start (Experimental)
ctest_update (SOURCE "${CTEST_SOURCE_DIRECTORY}")
ctest_configure (BUILD "${CTEST_BINARY_DIRECTORY}")
# builds *all* targets: Libs and Exes
ctest_build (BUILD "${CTEST_BINARY_DIRECTORY}")
# runs *all* tests
ctest_test (BUILD "${CTEST_BINARY_DIRECTORY}")
# submits everything all at once at the end
ctest_submit ()
```

After the changes:
```cmake
ctest_start (Experimental)
ctest_update (SOURCE "${CTEST_SOURCE_DIRECTORY}")
ctest_submit (PARTS Update Notes)

# to get CTEST_PROJECT_SUBPROJECTS definition:
include ("${CTEST_SOURCE_DIRECTORY}/CTestConfig.cmake")

foreach (subproject ${CTEST_PROJECT_SUBPROJECTS})
  set_property (GLOBAL PROPERTY SubProject ${subproject})
  set_property (GLOBAL PROPERTY Label ${subproject})

  ctest_configure (BUILD "${CTEST_BINARY_DIRECTORY}")
  ctest_submit (PARTS Configure)

  set (CTEST_BUILD_TARGET "${subproject}")
  ctest_build (BUILD "${CTEST_BINARY_DIRECTORY}" APPEND)
    # builds target ${CTEST_BUILD_TARGET}
  ctest_submit (PARTS Build)

  ctest_test (BUILD "${CTEST_BINARY_DIRECTORY}"
    INCLUDE_LABEL "${subproject}"
  )
# runs only tests that have a LABELS property matching
# "${subproject}"
  ctest_submit (PARTS Test)
endforeach ()
```

In some projects, more than one [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) step may be required to build all the pieces of the subproject. For example, in Trilinos, each subproject builds the `${subproject}_libs target`, and then builds the all target to build all the configured executables in the test suite. They also configure dependencies such that only the executables that need to be built for the currently configured packages build when the all target is built.

Normally, if you submit multiple `Build.xml` files to CDash with the same exact build stamp, it will delete the existing entry and add the new entry in its place. In the case where multiple [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) steps are required, each with their own [`ctest_submit(PARTS Build)`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) call, use the `APPEND` keyword argument in all of the [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) calls that belong together. The `APPEND` flag tells CDash to accumulate the results from multiple submissions and display the aggregation of all of them in one row on the dashboard. From CDash’s perspective, multiple [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) calls (with the same build stamp and subproject and `APPEND` turned on) result in a single CDash build.

Adopt some of these tips and techniques in your favorite CMake-based project:
- `LABELS` is a CMake/CTest property that applies to source files, targets and tests. Labels are sent to CDash inside the resulting xml files.
- Use [`ctest_submit(PARTS …)`](https://cmake.org/cmake/help/latest/command/ctest_submit.html#command:ctest_submit) to do incremental submissions. Results are available for viewing on the dashboards sooner. Don’t forget to use `APPEND` in your [`ctest_build`](https://cmake.org/cmake/help/latest/command/ctest_build.html#command:ctest_build) calls when submitting by parts.
- Use `INCLUDE_LABEL` with [`ctest_test`](https://cmake.org/cmake/help/latest/command/ctest_test.html#command:ctest_test) to run only the tests with labels that match the regular expression.
- Use `CTEST_BUILD_TARGET` to build your subprojects one at a time, submitting subproject dashboards along the way.
