### Basic programs

```bash
#! /bin/bash

A comment

echo "Hello, world!"
```

The #! symbol combo is called a shebang. It specifies the interpreter executing the file. When you write something like ```./<filename>``` in the terminal, the OS finds the program specified after the shebang and calls it, using ```./<filename>``` as argument 

For example, if you write ```./basic_exe``` in the terminal, the OS 
will instead call ```/bin/bash ./basic_exe```

The shebang has to be the first symbol in the file to work 
correctly

In bash scripts symbols denote comments

A bash script consists of a series of simple or compound 
commands. Simple commands (essentially) take the form of 
`<command_name> <arg1> <arg2> .. <argn>`

`<command_name>` can be a relative (if it begins with `./`) or 
absolute (if it begins with `/`) path to some executabe file
If it is neither of those things, bash instead searches for an 
executable file named `<command_name>` in one of the default 
search locations (the list of these is defined by the `PATH` 
environment variable

Arguments are strings separated by spaces. If your argument 
includes a space it has to be escaped, either by prepending it 
with a `\`, or by enclosing it in quotes 

```bash
#! /bin/bash

./utility/argcount 1 2 3
# 3

./utility/argcount "1 2 3"
# 1
```
A list of commands is a sequence of one or more commands 
separated by `;`, `&`, `&&`, or `||`, and optionally terminated 
by one of `;`, `&`, or a newline

### Command lists

A list of commands
```bash
command; command; command || command &
```

If two commands are separated by a `;` they are simply executed 
one after another
```bash
echo "Line 1"; echo "Line 2"
```

If two commands are separated by a `&&` the second command is 
executed only if the first one finished successfuly (with an 
exit code of zero)
```bash
echo "This line is seen" && echo "And so is this one"

./utility/buggy_program && echo "This line isn't seen"
```

If two commands are separated by a `||` the second command is 
executed only if the first one finished unsuccessfuly (with an 
exit code other than zero)
```bash
echo "This line is seen" || echo "But not this one"

./utility/buggy_program && echo "This line is seen"
```
This is typically used in constructions like `<command that may fail> || exit 1` to exit early if some step of the program fails

If a command is followed by a `&`, it is executed asynchronously
```bash
sleep 10 & echo "This is seen immediately"

sleep 10; echo "This is seen after 10 seconds"
```

### Compound commands

A compound command takes the form
```( <list>; )```
or
```{ list; }```
The spaces can be replaced by any number of any blank symbols 
(space, tab, newline, etc). The `;` can be replaced by a newline.
For historical reasons the spaces between `{}` and list are 
mandatory, and the spaces between `()` and list are optional

These are all valid compound commands
```bash
{ command; command; }

{ command
}

(command;)

(command
command
command
)
```
Commands inside `{}` are executed normally. Commands inside `()` are 
executed in a subshell. That means that bash creates a separate 
interpreter for these commands. As such, if it crashes (for 
example, if you execute `exit 1`), your main program will be 
unaffected. Also, variable assignments, redirections, `shopt` 
directives and suchlike inside `()` will have no effect outside of it

`for`, `while`, `if` and `case` constructs are also considered 
compound commands 

### Quoting (abridged)

Strings inside `''` are left exactly as they are. No word
splitting or other manipulations are performed 

Strings inside `""` are mostly left as they are. No word 
splitting is performed, most special characters are interpreted
literally, but operations with a more explicit syntax 
(like variable dereferencing) are still performed

Unquoted strings are left at Bash's mercy

```bash
variable=some_value

# $<varname> typically gets replaced with the 
# variable's value

# And ~ typically gets replaced with the path 
# to your home dir

echo '$variable and ~'
# $variable and ~
./utility/argcount '$variable and ~'
# 1

echo "$variable and ~"
# some_value and ~
./utility/argcount "$variable and ~"
# 1

echo $variable and ~
# some_value and /path/to/home/dir
./utility/argcount "$variable and ~"
# 3 
# (or more, depending on if /path/to/home/dir has spaces)

```



