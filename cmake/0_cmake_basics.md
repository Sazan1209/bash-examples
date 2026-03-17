### Cmake basics

Cmake is a program (and a programming language) designed to
facilitate C/C++ builds. Cmake builds targets (usually executables
or libraries) as defined in a CMakeLists.txt file (written in 
cmake, of course)

The main commands for working with cmake are as follows. 
To configure a project: 
```bash
cmake -S <source_dir> -B <build_dir> <options>
```
Where:

`<source_dir>` is the directory containing the 
CMakeLists.txt file.

`<build_dir>` is the directory which will be used to store
build artefacts and configuration files. This one can (and should) be reused
when rebuilding the same project. <b>Note:</b> in cmake
lingo, the build directory is known as the <i>binary directory</i>

`<options>` are various options and definitions (mostly definitions)
which control how the build is to be configured. To define
a variable named `varname` with the value `value` you,
should pass as argument `-Dvarname=value`

These values are cached and are preserved between rebuilds,
so you don't have to input them every time

The list of definitions can potentially be very large, so unless 
you're doing something basic, when first building a project
you should use `cmake-gui` to configure it properly
(or use your IDE, if it supports such a thing. 
Most profesional ones do)

To actually build a configured project, use

```bash
cmake --build <build_dir> --target <target_name>
```

`<target_name>` can be any of the targets being built by 
your project. If you omit the `--target` option, a special
target named `all` is built, which builds (almost) all
other targets

### Cmake lifecycle

Firstly, one thing to note is that cmake doesn't actually 
build your programm itself. Instead it writes configuration
files for a different program (called a <i>generator</i>)
which will then read these files and actually build your program

On Linux the default generator is Makefiles. Generators can
be specified when first configuring a project using the `-G`
option

The lifecyle of cmake builds consists of three stages
- Configuration (the part where configuration files are written)
- Generation (the part where <i>generator expressions</i> are evaluated. We won't talk about these)
- Build (the part where your project is built)

The first two stages are done by cmake itself when invoking
`cmake -S source -B build`, the last one is done by the generator
when invoking `cmake --build build`

### Cmake syntax

A cmake program consists of a series of commands. All cmake
commands take the form `command_name(arguments...)`.
CMake is not case sensitive to command names, 
but it is considered best practice to use lowercase commands. 
All whitespace (spaces, line feeds, tabs) is ignored except 
to separate arguments. Therefore, commands may span 
multiple lines as long as the command name and the
opening parenthesis are on the same line.

Arguments are separate by whitespace. To use an argument that
has spaces in it, enclose it in `""`. Alternatively you can
use a special syntax that takes the form `[====[ your string here ]====]`
You can use any amount of `=` inbetween the `[` (including 0) but you have to use the same
amount on each side.

Comments in cmake begin with `#`

Cmake has variables which can be set using the `set()` command
and invoked using the `${varname}` syntax. Variable names 
are case-sensitive

Cmake has branching logic, loops, custom functions and
macros, but we will not get into them

### Variables

All variables in cmake are strings. In some contexts they
can be interpreted by a program as a different type, but 
underneath they're still strings.

Cmake has three types of variables: regular variables,
cache variables and environment variables. 

Regular variables can be set using the following syntax

```cmake
set(<varname> <val1> <val2> ... <valN>)
```

Typically you will use only one `val` argument. 
Using multiple arguments results in them being concatenated
into a list, i.e. a string of the form `<val1>;<val2>;...;<valN>`.
More on lists later

To access a variable, as previously mentioned use `${varname}`

--------------------------------------------------

Cache variables are more tricky. They are preserved between 
reruns of cmake and are intended to be set by the user when 
configuring a project

A cache variable can be created using the `set()` command
using the following syntax

```cmake

set(<varname> <values...> CACHE <type> <docstring> [FORCE])

```


`<type>` has to be one of `BOOL` (boolean),
`PATH` (path to directory), `FILEPATH` (path to file),
`STRING` (string) or `INTERNAL` (not intended for manipulation
by the end user)

`<docstring>` is a string describing the variable

`<type>` and `<docstring>` are mandatory, although 
`<docstring>` can be empty. They do not affect cmake
behavoir in any way and are meant to provide clarification
for the end duser (i.e. if you configure your project 
using cmake-gui, the docstring and type will show up there)

--------------------------

When setting a cache variable, the set command will create
it if it doesn't exist, providing the default value `<values>`
and the given type and docstring, unless already present.

This means that if the user has already defined a value
for a given cache varible, or it has been defined elsewhere 
in the project, if will not be overwritten

If you do want to overwrite it (although you shouldn't)
you have to use the `FORCE` optional argument

A cache variable and a regular variable can share the same
name. You should avoid having this happens. CMake will generally prefer the regular variable, 
although not always. The behaviour can be buggy or difficult
to predict 

-----------------------------

Because setting a boolean cache variable is so common, cmake has the `option(<varname> <docstring> [value])` command
which is equivalent to `set(<varname> <value> CACHE BOOL <docstring>)`.
`<value>` defaults to OFF if unspecified

---------------


Environment variables can be accessed using the `ENV{<varname>}` syntax
They can be set using the `set()` command, but this 
can lead to unpreictable behaviour 




### Values

Although cmake variables are always strings, certain 
conventions exist on how to represent other types as strings

- Boolean: ON, YES, TRUE, Y (in any case) and non-zero numbers are considered
true. OFF, NO, FALSE, N, IGNORE, NOTFOUND, an empty
string, a string that ends in -NOTFOUND and zero are considered false. In cases
other than the above, cmake behaviour becomes complicated and
obtuse, so be careful

- Lists: a list is a sequence of values separated by (unescaped) `;`. 
Lists can be manipulated using the `list()` command, which 
we will not discuss

- Paths: cmake stores paths using forward slash, regardless
of the underlying OS. Paths can be manipulated using the
`get_filename_component()` and `file()`, or, after version
3.20, the combined `cmake_path()` command. We will not
discuss these.

- Numbers: are stored as one would expect. Math can be performed
using the `math()` command.


### Subdirectories

Typically, for convinience, CMakeLists.txt for complex project are split
into several files, each corresponding to some submodule
of a project. This behaviour is supported by the 
`include()` and `add_subdirectory()` commands

-----------------------------

`include(<file_path>)` includes the file specified by `<include_path>`.
The behaviour of this is (mostly) equivalent to just copy-pasting
the contents of the file in place of the `include()` command

Most commonly this command is used to include cmake <i>modules</i>,
i.e. bundles of cmake code which implement specific 
functions and features. For example, the FetchContent module
(which we will discuss later) implements the `FetchContent()`
function, which allows installing certain packages during 
build time

Because of this, if given a module name `<module>`, instead of a path
as an argument, the include command will automatically
search for a file named `<module>.cmake` in a list of 
locations.

Specifically, it will first go through all directories
listed in the `CMAKE_MODULE_PATH` variable, and then 
an internal cmake directory, containing certain built-in
modules

It is seemingly unspecified how cmake distinguishes between
a file path and a module name. On Linux the implementation
checks whether the argument begins with either `/` or `~`
and on Windows it checks whether the argument's second character is `:` (as in `C:`)
or if it starts with `\`

--------------------------------

```cmake

add_subdirectory(<sourceDir> [<binaryDir>]
  [EXCLUDE_FROM_ALL]
)

```

This command searches the directory `<sourceDir>` for 
a CMakeLists.txt file and includes it in the build.

Unlike the `include()` command, `add_subdirectory()`
creates a new <i> directory scope </i> for the included
code. This means that functions, macros and variables defined
inside the included file will not be visible in the 
parent scope (unless using special commands). Variables,
functions and macros from the parent scope will be 
inherited, but changes to them will not be visible in the
parent scope

An exception to this rule is targets. Targets defined 
in the child scope will be visible in the parent scope 
(with a couple exceptions we won't get into)

In addition, `add_subirectory()` creates a separate
<i> binary directory </i> for the new scope. 
What that means, is that cmake will create a new 
subdirectory inside the regular `build` directory,
containing the build artefacts for this specific part 
of the project

The name for the new binary directory can be specified 
explicitly, using the optional `<binaryDir>` argument.
If it is unspecified, cmake will just mirror the `<sourceDir>`
argument

For example, if I have a binary dir named `build` and a source dir 
named `project` and I run `add_subdirectory(some/path/to/subdir)`
cmake will also create the directory `build/some/path/to/subdir`

The optional `EXCLUDE_FROM_ALL` argument makes it such that
targets inside the included CMakeLists.txt will be excluded 
from `all`

### Directory variables

There are a number of built-in variables that allow one to
refer to various build-related directories

- CMAKE_SOURCE_DIR: the directory where the topmost
CMakeLists.txt resides. This is the directory you specify
as the value for the `-S` option when building.

- CMAKE_BINARY_DIR: same, but binary instead of source.
The value for the `-B` argument.

- CMAKE_CURRENT_SOURCE_DIR: the directory which corresponds
to the current directory scope. See `add_subdirectory()`. 
<b>Note:</b> usually cmake counts its relative paths
beginning with this directory.

- MAKE_CURRENT_SOURCE_DIR: the <i> binary </i> directory which corresponds
to the current directory scope. See `add_subdirectory()`. 
<b>Note:</b> sometimes cmake counts its relative paths
beginning with this directory.

- CURRENT_LIST_DIR: the directory where the current cmake file
resides 

- PROJECT_SOURCE_DIR: the source dir of the current project.
I.e. whenever the `project()` function is called, for all
files in the current directory scope, as well as its children
this variable will be set to the value of CMAKE_CURRENT_SOURCE_DIR
at the moment of `project()` being called.
