
---

In linux we have commnds 

### Wildcards

Running linux commands in the shell allows users to use its wildcard feature. Wildcard feature is not the part of `bin` utility but the part of linux shell. Wildcards are handled by the Shell (Bash/Zsh) before the command even runs.

|**Wildcard**|**Meaning**|**Example**|
|---|---|---|
|`*`|Matches **any number** of characters|`ls *.log` (All files ending in .log)|
|`?`|Matches exactly **one** character|`cat file?.txt` (Matches file1.txt, but not file10.txt)|
|`[ ]`|Matches any character **inside** the brackets|`ls [abc].txt` (Matches a.txt, b.txt, or c.txt)|
|`[!]`|Matches any character **not** inside brackets|`ls [!0-9].txt` (Matches any file not named with a number)|
So the command actually is transformed to diffrent command when when transfers control to actual binary. For example `ls *.txt`. bash will handle `*` itself and ls will get all the file names only. 

#### Filters - 

Ther are three important filters in linux environment. 

 - grep - **(Global Regular Expression Print):** Searches for specific text.
 - sort - Puts lines in alphabetical or numerical order.
 - uniq - Removes duplicate lines (usually requires the list to be sorted first).

These are often combined with `less,cat,tail or head`. The combination is done using `pipe`. 


#### grep

`grep` stands for **Global Regular Expression Print**. It scans text and prints lines that match a specific pattern.

Flags - 
- `-i` - ignore case
- `-v` - invert match - give everything that does not matches expression
- `-r` - Do recursive pattern match inside direcotries as well. 
-  `-c` - Gives only the count of files matched
- `-E` - extended regex

Example - 

```bash
grep [options] "regex" files
grep -i "failed" /var/log/auth.log
```

Gives out all lines having failed. 

#### sort 

It arranges lines of text in a specific order. By default, it sorts alphabetically.

Flags - 

- `-n` Sort by number value , by default it uses lexicographical sort.
- `-r` reverse
- `-k` sort based on some column 

```bash
sort -nk 2 logs.txt
```

#### uniq

`uniq` identifies and removes adjacent duplicate lines.**Critical Rule:** `uniq` only detects duplicates that are **next to each other**. This is why you almost always use `sort` before `uniq`.

Flags - 

- `-c` Tells you how many times each line appears
- `-u` - unique only - Only shows lines that were _not_ duplicated.
- `-d` -  Only shows the lines that _were_ duplicated.

#### cut

The `cut` command is your "surgical scalpel" in Linux. Its sole purpose is to extract specific sections (columns) from each line of a file or piped output. It is most effective when dealing with structured text like CSVs, log files, or system tables (like `/etc/passwd`).

IT has two modes - 

1. Cut by character

- **Single character:** `cut -c 5 file.txt` (Extracts only the 5th character).
- **Range:** `cut -c 1-10 file.txt` (Extracts the first 10 characters).
- **Multiple ranges:** `cut -c 1-5,10-15 file.txt` (Extracts characters 1-5 and 10-15).

1. Cutting by field

This is the most common usage. It works like a spreadsheet, where you define what separates the "cells" (the delimiter) and which "column" (the field) you want.
- **`-d`**: The delimiter (the character that separates columns). The default is a **Tab**.
- **`-f`**: The field number you want to keep.