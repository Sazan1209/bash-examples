### Variables
Bash variables have a name, value and attributes. 
Variables can be declared expilicitly, using the declare builtin
```bash
declare [<options>] <name>[=<value>]
```
or implicitly, just by assigning to the variable 
```bash
some_var=123
```
Note that you can't add spaces around `=`
```bash
# this is invalid
a = 123
declare var = 4
```
Attributes can be specified using the `declare` builtin either
during or after creation

```bash
# Specifies the "integer" attribute
declare -i new_variable
existing_var=12
declare -i existing_var
```

The various attributes are as follows:

`-a`: variable is an indexed array (more on these later)

`-A`: varibale is an associative array (more on these later)

`-f`: variable is a function 

`-i`: variable is an integer arithmetic evaluation (more on that below) is performed when the variable is assigned a value

`-x`: variable is <i>inherited</i>, meaning its value will be avalible for subsequent commands

There are other attributes, but they're mostly useless, so we'll not mention them

Variables can be dereferenced by using the `${<varname>}` syntax 
or just `$<varname>`. Note that in the latter case all alnum
characters after `$` are treated as part of `<varname>` which 
may lead to ambiguity
```bash
example="Hello, world!"
echo "$exmaple"
```
The variable doesn't have to be defined to be dereferenced. The 
value of an undefined variable is the empty string. This is very 
bad because **if you make a typo in a variable's name, the code will
execute incorrectly rather than throw an error**. This behavoir can
be disabled by running `set -u`

Bash finds the variable and substitutes its value inside the 
text of the program.
```bash
var=ex
echo Some t${var}t
# Some text

var=ch
e${var}o "Hello, World!"
# Hello, World!
```

Variable dereferencing is a type of bash parameter expansion 
(more on these later) known as Parameter Expansion. 

Bash automatically creates variables named 0...n, where n is the 
number of program arguments. Variable 0 contains the name of the
program and variables 1 through n contain the arguments
For example, if one calls `./dir/other/command 1 asd "sss xxx"` 
variables named `0`, `1`, `2`, `3` will have values 
`./dir/other/command`, `1`, `asd`, `sss xxx`

### Arithmetic expressions

Expressions inside `(())` and `$(())` (as well as in some other 
contexts) are evaluated as bash arithmetic expressions. Bash
arithmetic experssions mostly follow the same rules as in c, 
supporting all the same operators, including assignment operators

All calculations are preformed using 64-bit signed arithmetic 
(technically no, but who cares). 

Variables in arithmetic expressions can be dereferenced without
using `${}`. This sometimes leads to unexpected behavoir.  

```bash
var=2
(( other_var = (var *= 26) + 1 << 22 )) 
echo ${var}
# 52
echo ${other_var} 
# 222298112
```

The empty string evaluates to zero. Non-empty string `<str>` evaluates to 
`${<str>}`. This also sometimes leads to unexpected behavoir

```bash
foo=bar
bar=fnord
fnord=25

((foo++))
# here foo evaluates to `bar`, 
# which evaluates again to `fnord` which evaluates to 25

echo ${foo}
# 26
```

The `((<expr>))` syntax evaluates `<expr>` as an arithmetic expression 
and returns a return code based on the result (1, or error, if the 
result is zero, and 0, or ok otherwise).

This behavoir is useful for conditional constructs like `if` and 
`while` (more on that later)

The `$(<expr>)` syntax evaluates `<expr>` as an arithmetic expression
and the result is then substituted for the original expression

```bash
echo $((1+1))
# 2
```

This is another type of expansion, known as Arithmetic Expansion

### Boolean expressions

Bash doesn't have a boolead data type. Instead it uses
program exit codes. A program that exits with an error
exit code (i.e. non-zero) evaluates to false. A program
that exits with an OK exit code (i.e. zero) evaluates to 
true.

This behavoir may seem bizarre, but it can be very convinient

Bash evaluates various boolean expressions using the `[[ <expr> ]]`
syntax (there is an older `[]` "syntax", but it's outdated
so we won't discuss it). Note that everything inside `[[]]` should 
be separated by spaces, because spaces are mandatory 
(except when they aren't) .

Unlike in arithmetic expressions, variables (generally)
have to be expilictly dereferenced using `$` or `${}`

Expressions consist of primaries, boolean operators 
`!`, `||`, `&&` and brackets `()`

The boolean operators behave the same as in c, except 
for spaces.

```bash

[[ a == a || b == c ]]
# true

[[ !a = b ]]
# invalid syntax. Outputs god knows what
# There should be a space after the !

[[ ! a = a ]]
# false

```

The primary boolean expressions are as follows

`<string>` - true if the string is non-zero

`-e file` - true if `file` exists

`-d file` - true if `file` exists and is a directory

`-d file` - true if `file` exists and is a directory

`-f file` - true if `file` exists and is a regular file

`-h file` - true if `file` exists and is a symlink

`file1 -ef file2` - true if `file1` and `file2` refer to the same 
underlying memory

`file1 -nt file2` - true if `file1` is newer (according 
to modification date) than `file2`, or if `file1` exists
and `file2` does not.

`file1 -ot file2` - true if `file1` is older than `file2`,
or if `file2` exists and `file1` does not.

`-v varname` - True if `varname` is set (has a value)

`<`, `<=`, `>`, `>=` - Compares strings **lexicographically**

`=`, `==`, `!=` - Tests strings for equality. If the left side is 
contains an unqoted pattern, Pattern Matching is performed 
(more on that later)

`-lt`, `-le`, `-gt`, `-ge`, `-eq`, `-ne`. Left and right side
are evaluated as arithmetic expressions (with all that brings)
and the compared (Less Than, Less or Equal, Greater Than, 
Greater or Equal, EQual, Not Equal)

Other primaries exist, but they are mostly useless, so we'll 
not cover them






