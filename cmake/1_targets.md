### Targets

The main object you will be working with are cmake targets
A cmake target is (typically) some kind of file/object that
is built by the project. This may be an executable, a library,
or may be purely virtual, as is the case with custom targets
or interface libraries (more on that later)

Targets may be created via the `add_executable()`
`add_library()` or `add_custom_target()` commands.
More on that later

A target has <i>properties</i> and <i>dependencies</i>. 

------------------------------------

Target properties are variables (almost always strings)
that are defined on a per-target level. Properties control
how the target is compiled and linked. 

Target properties are separated into what we're going to call
personal properties and inherited properties. 
The proper cmake name for these are <i>build specifications</i> and <i> usage requirements </i>
correspondingly

Personal properties affect the way this specific target 
is built, inherited properties affect the way it's descendents
(i.e., other targets which link to this target via the
target_link_libraries() command) are built

The effective value of a given property during compilation
is calculated by combining the personal property of the target
and the inherited property from all it's parents, including
transitive ones 

--------------------------

Target dependencies are other targets and files which have to 
be built before this target can start building. 

Cmake supports two types of dependency. I have been unable 
to find proper terms for them in the documentation, so we'll call 
them <i>weak</i> and <i>strong</i> dependencies

Weak dependencies are targets and files which need to be built
before the current target.

Strong dependencies are targets, which have to be older than 
the current target for the current target to be considered valid

The cmake lingo for this concept is targets being <i>out-of-date</i>
For example, if an executable is built from a source file, and the source file
is updated, the executable needs to be rebuilt. 

In other words, 
to be considered up-to-date a given target/file has to be
newer than its strong dependents
and all its strong dependents have to be up-to-date

Weak dependents of a target do not have to be up-to-date
for a target to be considered up-to-date, they only have to 
exist.

------------

When you build a given target in cmake, it is then brought up-to-date,
creating all of its weak dependents, if they didn't exist previously
and all of its strong dependents are brought up-to-date. Then, if need
be, the target itself is rebuilt.

These concepts will become more important when we get to custom targets and such

### Creating targets

Here we'll look at the `add_executable()` and `add_library()`
leaving `add_custom_target()` for much later

`add_executable()` has the following syntax

```cmake
add_executable(
  <name>
  [EXCLUDE_FROM_ALL]
  <sources...>
)
```

Running this command creates a target named `<name>` which
builds and executable named either `<name>` (on linux/macos) 
or `<name>.exe` (on windows) from the given sources. 

By default, the new target will be made a strong dependecy
of `all`, `unless EXCLUDE_FROM_ALL` is specified

`<sources>` are files which are required to build. They can be specified
using relative or absolute paths. Relative paths are 
calculated starting from the `CURRENT_SOURCE_DIR` directory

You only have to specify files which actually need to be compiled
(usually the .c/.cpp files). All files specified as sources will be
considered strong dependencies of the target. Cmake will
also detect header files, `include`-ed inside the `.cpp` files
and consider them strong dependencies as well

Only `.c`/`.cpp` files will actually be compiled, although other files
can be included in the source list

--------------------

`add_library()` has the following syntax
```cmake
add_library(
  <name>
  [<type>]
  [EXCLUDE_FROM_ALL]
  <sources...>
)
```

Running this will create a library object with the appropriate suffix/prefix
for a given system and library type (for example, `lib<name>.so` for a shared library on Linux)

Same comments apply to sources and `EXCLUDE_FROM_ALL` for libraries as for executables

The `<type>` field is more interesting. It can be one of 
`STATIC`, `SHARED`, `MODULE`, `OBJECT`, `INTERFACE`.
If the type is unspecified, cmake defaults to Static, or, 
if the `BUILD_SHARED_LIBS` is set to true, `SHARED`

Static and shared libraries are staticly and dynamicly linked libraries correspondingly.
Module and object libraries are fairly exotic, so we'll not discuss them.
Interface libraries are entirely virtual. They do not compile into anything
and are only used to propagate usage requirements. An example of where such a thing
would be useful is a header-only library. It doesn't have any compiling done, but 
libraries which link to it still need to be able to find the location of the header 
files, which can be done through cmake usage requirements

### IMPORTED AND ALIAS 

In addition to the above, cmake allows the creation
of imported and alias targets

Alias libraies or executables can be defined using the following syntax

```cmake
add_library(
  <name> 
  ALIAS 
  <target>
)

add_executable(
  <name> 
  ALIAS 
  <target>
)
```

Alias targets are read-only, meaning they cannot be modified
but are otherwise just aliases for the original target.

------------------------------

Imported libraries or executables can be defined using the
following syntax

```cmake
add_executable(
  <name>
  IMPORTED 
  [GLOBAL]
)

add_library(
  <name>
  (STATIC | SHARED | MODULE | OBJECT | UNKNOWN)
  IMPORTED
  [GLOBAL]
)

```

Imported targets are used to 
include pre-built libraries and executables 
to be used as cmake targets

The actual path to the library or executable file
has to be specified using the `IMPORTED_LOCATION` target
property.

Unlike all other types of target, imported targets are not 
global by default and the `GLOBAL` option has to be specified
for them to be availible outside the current directory scope.

--------------------------

Imported and alias libraries are not useful by themself
and are only really used when creating a project that
is meant to be consumed by other cmake projects.

The `STATIC`, `SHARED` and so on options for imported libraries
are used to specify the type of the imported library. If 
`UNKNOWN` is specified, cmake will try to determine the type
by itself.

### Properties 

Properties are a special type of variable that exist independently 
for different objects, depending on the type of property.

We've already seen target properties, which exists on a per-target
basis, but now we'll talk about properties more broadly.

The most general syntax for working with properties
is 

```cmake 

set_property(
  <entitySpecific>
  [APPEND | APPEND_STRING]
  PROPERTY <propertyName> <values...>
)

```

Where `<entitySpecific>` is one of 

```
GLOBAL
DIRECTORY [dir]
TARGET targets...
SOURCE sources... # Additional options with CMake 3.18
INSTALL files...
TEST tests...
CACHE vars...
```

for global, directory, target and etc. properties correspondingly.

Reading properties may be accomplished with the following syntax

```cmake
get_property(
  <resultVar>
  <entitySpecific>
  PROPERTY <propertyName>
)
```

The value of the property is placed inside the `<resultVar>`
variable.

-------------------------

Global and cache properties are fairly exotic, so we won't discuss them.

Test and install properties are only relevant when using CTest or the `install()` command,
which we won't discuss either.

--------------------------------

### Target properties

Target properties are the most common type of property. Typically 
they control how a given target is compiled. As already discussed,
target properties are separated into personal and inherited properties
(inherited properties always have the same name as the corresponding 
personal property, except with `INTERFACE_` prepended)

The most important target properties are as follows.

- INCLUDE_DIRECTORIES (INTERFACE_INCLUDE_DIRECTORIES).
Controls where the preprocessor will search for headers to `include`.

- COMPILE_OPTIONS (INTERFACE_COMPILE_OPTIONS). 
Contains the options which will be passed to the compiler when 
building the target.

- COMPILE_FEATURES (INTERFACE_COMPILE_FEATURES)
Contains the compiler features which are required when 
building the target

- COMPILE_DEFINITIONS (INTERFACE_COMPILE_DEFINITIONS)
Contains the preprocessor `DEFINE`-s, which will be
added to the code

- LINK_LIBRARIES (INTERFACE_LINK_LIBRARIES)
Contains the libraries which will get linked
to the target

- SOURCES (INTERFACE_SOURCES)
Contains the sources used to build the target.

- LINK_OPTIONS (INTERFACE_LINK_OPTIONS)
Contains the options used for linking the target.

All these properties have an associated 
setter command named `target_<property_name>`.
**It is strongly preffered you use this setter** instead of
manipulating the property directly.
The syntax for all setter commands is as follows.

```cmake
target_xxx(
  <target>
  {INTERFACE|PUBLIC|PRIVATE} <value>...
  [{INTERFACE|PUBLIC|PRIVATE} <value>...]...
)
```

Values marked with `PRIVATE` are added to 
the personal property. Values marked with `INTERFACE`
are added to the inherited property. `PUBLIC` values 
are added to both.

For example, to make a target and all its consumers
be compiled with the `-WALL -WEXTRA` options, one should
write `target_compile_options(<target_name> PUBLIC -WAll -WExtra)`

Some additional target properties are

- RUNTIME_OUTPUT_DIRECTORY
Used for executables on all platforms and DLLs on Windows.

- LIBRARY_OUTPUT_DIRECTORY
Used for shared libraries on non-Windows platforms.

- ARCHIVE_OUTPUT_DIRECTORY
Used for static libraries on all platforms and import libraries associated with DLLs on Windows.

- VERSION
Version number of a shared library target.

- SOVERSION
ABI version (or major version) number of a shared library target.

- BUILD_RPATH
A semicolon-separated list specifying runtime path (RPATH) entries to add to binaries linked in the build tree (for platforms that support it).

- INSTALL_RPATH
Same as the build rpath, but used after installation

### Directory properties

Typically, directory properties are
used as the base value for corresponding 
target properties. Some directory properties and their 
corresponding setters are

- INCLUDE_DIRECTORIES (setter `include_directories()`)
- COMPILE_DEFINITIONS (setter `add_compile_definitions()`)
- COMPILE_OPTIONS (setter `add_compile_options()`)
- LINK_OPTION (setter `add_link_options()`)

Because these options provide the base value
for target properties, they only have an effect on 
targets created afterwards. An exception to this
are the `include_directories` and `add_compile_definitions`
functions, which affect previous targets, but only in 
the current directory scope.

For example here
```cmake
add_executable(exe source.cpp)
add_subdirectory(dir)

include_directories(includeDir)

add_executable(exe2 source2.cpp)
```
the includeDir applies to exe and exe2
but not any targets created insidde 
dir, as thats a different scope.

### SOURCE PROPERTIES

Sometimes, although rarely, one wishes to manipulate
the compilation on the level of individual files.
This is where source properties come into play.
Some source properties are

- COMPILE_DEFINITIONS
- COMPILE_FLAGS
- COMPILE_OPTIONS
- INCLUDE_DIRECTORIES
- LANGUAGE

Most of these you've already seen, so we won't explain 
them. They override the corresponding target property,
which allows for precise control.

The language property controls which language the file
is in, such as `CXX` or `C`. If it is unset, cmake
determines it based on the extension.



