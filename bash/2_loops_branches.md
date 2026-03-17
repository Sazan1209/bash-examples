### Loops and branches
Loops and branches in bash work the same as they do in 
other languages


while cycle syntax:
`while <test-commands>; do <consequent-commands>; done`


The ; can be replaced with newline. Spaces can be replaced with 
any number of any blanks

```bash
while command; do command; done

while
command; do 
  command
done
```

if-else works as expected. The syntax is as follows
```bash
if test-commands; then
  consequent-commands;
[elif more-test-commands; then
  more-consequents;]
[else alternate-consequents;]
fi
```

Newlines are optional and can be replaced with any blanks
```bash
if command; then command; fi

if command
then
  command
else
  command
fi

if command
then
  command
elif command
  command
else
  command
fi
```

Bash has c-style for loops

`for (( expr1 ; expr2 ; expr3 )) [;] do commands ; done`

`expr{1,2,3}` are evaluated as arithmetic expressions. 

```bash
for (( i=1 ; i < 20 ; ++i )) do command ; done

for (( i=1 ; i < 20 ; ++i )) 
do 
  commands
done
```

Bash has range-based loops

`for <varname> in <words...>; do <commands> ; done`

```bash

for name in Alice Bob Carol Dwayne
do
  echo "Hello, $name!"
done

```



