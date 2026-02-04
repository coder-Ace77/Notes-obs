
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

### <font color="#2DC26B">System info</font>

- top/htop - Shows real-time CPU, Memory, and Process usage.
- df -h -  Shows Disk Free space in a human-readable format.
- **`free -m`**: Shows available Memory in Megabytes.
- **`ps aux`**: Lists every running process on the system
- **`kill -9 <PID>`**: Force-stops a process using its Process ID.

Note ps aupports the sorting by default `ps aux --sort=-%mem | head -n 5`

Some netowrking commands - 

- ip addr - shows ip address and network interfaces
- ping - Checks if a remote host is reachable.
- **`curl` / `wget`**: Downloads files or interacts with URLs directly from the CLI.
- **`ssh`**: Securely connects to a remote Linux machine.

### find

The `find` command is one of the most powerful—and occasionally intimidating—tools in Linux.

`find [where to look] [criteria] [what to do]`

Search by name is the most common use case. Note that using quotes around the filename is a "best practice" to prevent the shell from getting confused by wildcards. `find . -name "test.txt"`. Case insensitive ``find . -iname "test.txt"` (finds Test.txt, TEST.txt, etc.)` wildcards ``find /home/user -name "*.log"` (finds all files ending in .log)`

We can also search by type and size - `find /etc -type f`: Find only **files**. `find /var/log -size +50M`: Find files **larger** than 50MB.

find can perform some action on the files it founds using `-exec`. Eg

**Find all .log files and delete them** `find /tmp -name "*.log" -exec rm {} \;`

After -exec we write the command `{}` stands for placeholder and `\` states that command has finished. 


### Pipe

The pipe symbol (`|`) is arguably the most powerful character in the Linux CLI. It allows you to take the **Standard Output** (stdout) of one command and feed it directly into the **Standard Input** (stdin) of another.

### sed

Unlike a traditional text editor (like Notepad, Vim, or VS Code) where you open a file, move a cursor, and type changes, `sed` is a **non-interactive** editor. It takes text from a file or a pipe, transforms it according to your instructions, and spits it out. It is the king of "search and replace" in the Linux world.

`sed` works by reading text line-by-line into a temporary storage space called the **pattern space**. It applies your commands to that line, prints the result, and then clears the space for the next line.

The most common way to use `sed` is for substitution:

```bash
sed 's/old_word/new_word/' filename
```

**`old_word`**: What you are looking for (Regex is allowed here!).

By default, `sed` only changes the **first** occurrence on each line. To change every instance, add the `g` (global) flag:

```sh
# Change every instance of "Windows" to "Linux"
sed 's/Windows/Linux/g' document.txt
```

`i` flag can be make changes and save it to file.

```sh
# This modifies the file permanently
sed -i 's/localhost/127.0.0.1/g' config.php
```

| **Command**        | **Action**                             |
| ------------------ | -------------------------------------- |
| `s/find/replace/`  | Substitute the first match per line.   |
| `s/find/replace/g` | Substitute **all** matches per line.   |
| `/pattern/d`       | **Delete** lines matching the pattern. |
| `-i`               | **Edit** the file directly (in-place). |
| `-n ... p`         | **Print** only specific lines.         |

### Watch 

It clears screen and runs a given command after intervals defult 2s.

```sh
watch [options] commnad
```

<font color="#2DC26B">Options</font>

- `-n` Sets the interval in seconds
- `-d` HIghilight difference between updates
- `-t` turn of the header.

