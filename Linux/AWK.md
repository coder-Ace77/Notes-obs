
---

awk is very powerfull text processing tool it can either take values from the files or from other commands using pipe

```bash
awk 'pattern { action }' file

command | awk 'pattern { action }'
```

If no pattern is given action is executed for every line

So basically the pattern will match which lines to be executed and then inside action we can process the line line is divivded into words separated by spaces. 

| Symbol  | Meaning           |
| ------- | ----------------- |
| `$1`    | 1st word          |
| `$2`    | 2nd word          |
| `$3`    | 3rd word          |
| `NF`    | number of fields  |
| `$NF`   | last field        |
| `$NF-1` | second last field |
```bash
awk '{ print $2 }' file
```

Prints second word of every line. `print` is used to print some word. We can also do other things as well. We can also change field separator

`-F:` → split on `:`
```bash
awk -F: '{ print $1 }' file
```

Optionally we have begin and end blocks which run before and after processing all the lines. 

```bash
awk '
BEGIN { print "Start" }
{ print $1 }
END { print "Done" }
' file
```

## Action part

We can do multiple things with action

most simple is to print the line. By default the action is `print $0` where `$0` represents the whole line. 

Print lines containing `error`

```bash
awk '/error/'

awk '$3 > 100'
```

print can print multiple things by separation with , . Also we can use if and else and all the rules of condtion are followed as cpp rules. 

```bash
awk '{ 
    if ($2 > 50) 
        print "High:", $2
    else
        print "Low:", $2
}'

```

| Variable | Meaning                |
| -------- | ---------------------- |
| `NR`     | line number            |
| `NF`     | number of fields       |
| `FS`     | field separator        |
| `OFS`    | output field separator |
| `FNR`    | line number _per file_ |
## String and arithematic operations

```bash
awk '{ full = $1 $2 }' # concat
awk '{ print length($0) }' # length
awk '{ print substr($1, 1, 3) }' # substr

# arithematic
awk '{ print $1 + $2 }'
awk '{ sum += $1 } END { print sum }' # getting sum so observe that sum remained throughout scope
```


AWK has associative arrays(hashmap)

```bash
awk '{
    count[$1]++
} END {
    for (k in count)
        print k, count[k]
}' file.txt
```

- No need to declare
- Dynamically sized
- Automatically created on use
- Keys iterate in **random order**

Multiline awk - we can write multple action lines for each awk. begin and end can have multiple things. 


```bash
awk '
{
    a = $1
    b = $2
    print a + b
}
' file

awk '
$3 > 50 {
    print "Name:", $1
    print "Score:", $3
}
' file

```

We can have c style for loop 

```bash
awk '
{
    for (i = 1; i <= NF; i++) {
        print "Field", i, "=", $i
    }
}
' file

# for each 
awk '{count[$1]++}
END {
    for (k in count) {
        print k, count[k]
    }
}
' file

```

We can declare variable in a block and use them in same or later blocks. Also each line can have multiple pattern or conditions. 

```bash
awk '
BEGIN { threshold = 50 }

$3 > threshold {
    high[$1]++
}

$3 <= threshold {
    low[$1]++
}

END {
    print "High scorers:"
    for (k in high) print k, high[k]

    print "Low scorers:"
    for (k in low) print k, low[k]
}
' file
```

Variables can also be declared insdie loop

```bash
awk '
{
    total = 0
    for (i = 1; i <= NF; i++) {
        total += $i
    }
    avg = total / NF
    print "Average:", avg
}
' file
```