
---

Basic bash program

```bash
#!/bin/bash

# bash program here
```

### Variables

 Bash has only two kinds of variables 
 - Strings
 - integer

Normally all the input and output is dealt in strings however we can convert the string into integer uatomatically at time of operations. 
A variable is just a container and between = there must be no spacing.

```bash
#!/bin/bash

name="Adil"
x=1
```

Reading and consoling input / output and accessing the variable. The value of variable is accessed by using `$` operator or `${}` operator.
Also `${} or $` both work inside string as well. 

```bash
#!/bin/bash

name="Adil"
echo "Helllo $name"
echo "$name" # both will work
echo "${name}123" # {} is used to remove ambiguity

# taking input example

# read -options variable
read name
echo "Hello ${name}"
```

Options in read - 

- p : display prompt before input 
- s : quiet mode like password
- n: read only n chars

Any command of linux will run on bash however one may also get the output in variables

```bash
today=$(date)
# var=$(command)
```

## String operations

```bash

# length
s="Adil"
echo ${#s} # length by # operator

# substring
echo ${s:0:5} # ${var:start:length} length is optional and by defalut is till end

# string replacement

s="foo bar foo"
echo ${s/foo/baz} # replace first
echo ${s//foo/baz} # replace all
```

String comparison

```bash
a="hello"
b="world"

if [[ "$a" == "$b" ]]; then
    echo equal
else
    echo not equal
fi
```

Not equal

```bash
[[ "$a" != "$b" ]]
```

Contains check 

```bash
s="hello world"

if [[ $s == *world* ]]; then
    echo yes
fi
```

Empty non empty

```bash
[[ -z "$s" ]]   # empty?
[[ -n "$s" ]]   # not empty?
```

### Integer operations

Math can be done using:
- `$(( ))` — arithmetic expansion (returns value)
- `(( ))` — arithmetic command (true/false)
- `let` — older syntax (avoid)

```bash
a=10
b=3

echo $((a+b))   # 13
echo $((a-b))   # 7
echo $((a*b))   # 30
echo $((a/b))   # 3
echo $((a%b))   # 1

((a++))
((b--))
((++a))
((--b))

((a = 5 + 7)) # any assignent can be done here
echo $a    # 12
```

Comparison 

```bash
if (( a > b )); then
    echo greater
fi

for ((i=0; i<5; i++)); do 
    echo $i
done
```

### File operations

File existance
```bash
if [ -e file.txt ]; then
    echo "exists"
fi

# regular file
[ -f file.txt ]

#directory
[ -d mydir ]

# file is readable writable or executable
[ -r file.txt ]
[ -w file.txt ]
[ -x file.txt ]

```

| Flag | Meaning                    |
| ---- | -------------------------- |
| `-e` | exists (file or directory) |
| `-f` | a regular file             |
| `-d` | directory                  |
| `-r` | readable                   |
| `-w` | writable                   |
| `-x` | executable                 |
| `-s` | file exists and size > 0   |
| `!`  | NOT operator               |
Reading file

```bash
cat file.txt
```

Read can be used to read file line by line 

`-r` avoids backslash issues
`IFS=` avoids trimming spaces

```bash
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

Reading entire file into variable

```bash
content=$(<file.txt)
echo "$content"
```

Writing and appending

```bash
echo "Hello" > file.txt
echo "Next line" >> file.txt

ls > out.txt
date >> log.txt
```

Creating files and directories

```bash
touch file.txt
mkdir mydir
mkdir -p a/b/c      # create parent dirs automatically

# copying files

cp file1 file2
cp -r dir1 dir2     # recursive

# moving 
mv old.txt new.txt
mv file.txt dir/

# deletion

rm file.txt
rm -r mydir
rm -rf important    # force delete (dangerous)
```

Certain patterns like `*.txt` can be used to list all file names with .txt etc. So with this we can list all the files.

```bash

for f in *.txt; do
    echo "$f"
done

```

## if-else

Bash supports different **test syntaxes**:

- `[` `]` → old test
- `[[` `]]` → modern, safer
- `(( ))` → arithmetic
- `test` → old command

Best is `[[]]` 

```bash
if [[ condition ]]; then
    commands
fi

if [[ $age -lt 18 ]]; then
    echo "minor"
elif [[ $age -lt 60 ]]; then
    echo "adult"
else
    echo "senior"
fi

# string comparison
[[ $a == $b ]]
[[ $a != $b ]]
[[ -z $a ]]   # empty?
[[ -n $a ]]   # not empty?
[[ $a == *hello* ]]   # contains

#numberic commparison
[[ $x -eq $y ]]
[[ $x -gt 10 ]]

# file check 
if [[ -f "data.txt" ]]; then
    echo "file exists"
fi

# (()) type will support the arithematic operations as well. 
```

## Loops

Looping over a list

```bash
for x in 1 2 3 4; do
    echo $x
done

for f in *.txt; do
    echo "$f"
done

# c style
for ((i=0; i<5; i++)); do
    echo $i
done

# looping over array
arr=("a" "b" "c")

for x in "${arr[@]}"; do
    echo $x
done

# while loop

count=1
while [[ $count -le 5 ]]; do
    echo $count
    ((count++))
done

# reading file line by line
while IFS= read -r line; do
    echo "$line"
done < file.txt

# we can add break statement with a condtition

i=1

while IFS= read -r line; do
    echo $line
    ((i++))
    [[ $i -gt 10 ]] && break
done < file.txt

# we can pass a line with <<< operator as well 

print_second_words() {
    while IFS= read -r line; do
        set -- $line
        echo "$2"
    done <<< "$1" 
}

text="hello world here
this is cool
bash scripting rocks"

print_second_words "$text"

```

Note -  

1. -- marks the end of options it says what ever after is not a flag set -- $line will set the string words as dollar param So that is $2 reperensents second  word. 
### functions

```bash
myfunc() {
    echo "Hello"
}

myfunc # calling unction
myfunc arg1 arg2
```

Accsssing args 

|Symbol|Meaning|
|---|---|
|`$1`|first argument|
|`$2`|second argument|
|`$@`|all arguments|
|`$#`|number of args|
return keyword can not be used to return a valu instead it is used to return exit quotes. To return a value we echo it and then use command substitution

```bash
greet() {
    echo "Hello, $1"
}

msg=$(greet Adil)
echo $msg

```

To create local variables inside bash
```bash
calc() {
    local x=10
    echo $((x * 2))
}

calc
echo $x     # empty, because local
```

We can write functiions in other files and import them 

```bash

# in util 
log() {
    echo "[$(date +"%H:%M:%S")] $1"
}


# main
source utils.sh

log "running"

```

### Sort command 

```bash
sort options filename.txt
sort -r filename.txt
sort -n numbers.txt # sort based on numbers instead of numbers

sort -k 2 filename.txt #sort based on kth col 
sort -u filename.txt

sort -t ',' -k 2 filename.csv # -t changes the delimiter
```

### Passing args to script

We can pass argment as space separated list. `./script.sh arg1 arg2` the shell automatically assigns these values to numeric variables.

`$0` is name of script itself, `$1 to $9` first 9 args and `$(10)` for tenth arg onwards. `$#` is used to find length of args and `$@` for all args as list. 

### Arrays and for loops

Bash supports **one-dimensional** indexed arrays.

Defining arrays - 

```sh
# Method 1: Literal assignment
servers=("web01" "web02" "db01" "app01")

# Method 2: Index assignment
servers[4]="backup01"
```

Accessing array elements requires curly braces `${}`. If you omit them, Bash interprets `$servers` as only the first element. For example to get ith index `${servers[1]}` array length `${#servers[@]}`. Now we can also assign non conscutive numbers. 

```sh
servers={}
server[100]="adil"
```

now in this case we may need to get list of indexes and can be get by  `${!servers[@]}`. Always use **double quotes** around arrays (e.g., `"${servers[@]}"`) to prevent items with spaces from being split into multiple arguments.

The `for` loop is used to iterate over a list of items. In Bash, there are two primary styles:

`in` style 

```sh
# Iterating over a hardcoded list
for fruit in apple banana cherry; do
    echo "I like $fruit"
done

# Iterating over an array
for srv in "${servers[@]}"; do
    echo "Checking status of: $srv"
done
```

`c` style

If you need to run a loop a specific number of times or use an incrementing index, use the double-parentheses syntax.

```sh
# format: (( start ; condition ; step ))
for (( i=0; i<10; i++ )); do
    echo "Count is $i"
done
```

