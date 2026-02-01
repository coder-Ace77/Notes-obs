
---

Linux treats everything from your documents to your hard drive and even your mouse as a **file**. It’s a beautifully organized, hierarchical structure that looks like an inverted tree. At core linux uses virtual filesystem`(VFS)`. This is a software layer that lets the kernel talk to different types of file systems (like NTFS, FAT32, or Ext4) using the same set of commands. Basically VFS acts as a layer between different setfo devices and let the kernal talk to all of them using file like sytax commands. 

Unlike Windows, which identifies files by name, Linux identifies files by an **Inode number**. An inode is a data structure that stores everything about a file (metadata, permissions, owner) _except_ its actual name and the data itself.

#### Root directory

Linux file system has one root `/` and any files or devices need to get monted on this tree at some level. 

- bin - This folder has binaries and executables such as `ls , cp etc`.
- boot - Contains the files needed to start the OS, including the Linux Kernel and the GRUB bootloader. It is here the linux compressed binary is available. 
- dev - Devices directory - This contains "device files." For example, `/dev/sda` represents your hard drive. You don't "open" these; the system uses them to talk to hardware.
- etc - The nerve center for **configuration files**. If you want to change how a program behaves system-wide, you usually edit a file in here. So for example the `nginx` config files live here. 
- home - Where users keep their personal files. Each user gets a subfolder (e.g., `/home/username`). This is the only place a normal user can write files by default.
- bin - Contains shared library images (the Linux version of `.dll` files) needed by the binaries in `/bin` and `/sbin`.
- media - The temporary mount point for removable media like USB sticks or external hard drives.
- mnt - Historically used for manually mounting temporary filesystems.

- opt - Where "add-on" or third-party software is installed (like Google Chrome or Discord).

- proc - A "pseudo" filesystem. It doesn't exist on your disk; it exists in memory. It contains info about running processes and system resources. From proc we can read files like /proc/cpuinfo and get info about cpu. These are not real files but kernel creates them on the fly. 

- root - This is the home directory for the **root user** (the superadmin). It is separate from `/home` for security.
- sbin - Similar to `/bin`, but contains commands intended for the system administrator (e.g., `fdisk`, `reboot`).
- tmp - A place for applications to store temporary files. Usually cleared every time you reboot.
- usr - Contains the majority of user utilities and applications. It has its own `/usr/bin`, `/usr/lib`, and `/usr/share`. 
- var - For files that change frequently, like **system logs** (`/var/log`) and database files.

If you can see linux kernel code is not present anywhere in the directories. Essentially linu kernel is monolithic and gets loaded at boot time and from there on remains entirely in memory. Howeer its compiled binary file can be found in /boot. Here you also find the grub , memory test files and Recovery files. 

In Linux, because everything is a file, **permissions** are the gatekeepers that decide who can see, change, or run those files. It’s the backbone of Linux security it’s why a regular user can’t just accidentally delete the entire operating system.

When you run the command `ls -l` in a terminal, you’ll see a string of characters like `-rwxr-xr--`. This is the DNA of the file's permissions.

There are three kinds of users - 

- Owner (u) - Usually the person who created file
- Group (g) - A collection of users who share certain access
- Others (o) - Everyone else on the system

Similarly there are three permissions which can be given to any user - 

- read - r 
- write - w
- execute - x

In the string `-rwxr-xr--` we have following -  

 - The first character `-` means it is file while `d` means its directory. Character 2-4 are for permissions to the owner, 5-7 for group and 8-10 for others. This string can also be written as numeric system - `read = 4 , write = 2 and exec = 1`. In this format entire string can be wirtten as the three digit number. With each number representing the permissions about one user and following addititive format meaning `7 =  4+2+1` means read + write + execute. 
- To change the permission we use `chmod new_perm file`. To change owner we use `chown new_user file`.

The reason you can't just go into `/etc/shadow` (where passwords are kept) is that its permissions are usually `600` and it's owned by **root**. Without using `sudo` (SuperUser Do) to temporarily act as the root user, the system will simply say "Permission Denied."

When you run a file whether it's a Python script, a compiled C program, or a simple bash script the computer doesn't just "run the code." It creates a **Process**, and that process inherits its "identity" (and thus its permissions) from the user who started it.

Every process in linux has something called EUID(effective user id). This is the user id of user who runs it. Now if a process runs another process then EUID is inhierited. This effectively means process can inherit the permissions of users running it. 

`sudo` which means substitute user do **`sudo`** allows you to run a command as _any_ user (though it defaults to the root user if you don't specify one).

```
sudo -u bob python script.py
```


When you run `sudo python example.py`, you are telling the system: _"Run this process with the identity of the **Root** user."_ * The Process UID becomes `0` (Root). 

SUID bit  - There is a special permission bit called **SUID (Set User ID)**. If a file has this bit set, it runs with the permissions of the **file owner**, not the user who launched it.

### Commands 

Following are the important linux commands - 

`pwd` tells  you about the current folder.

`cd` is used to change directory `..` to ge up hierearchy. and `-` to ge to prev point.   

`ls` tells about the folder you are currently in 

```bash
ls [options] [directory]
```

| **Option** | **Description**                                                                          |
| ---------- | ---------------------------------------------------------------------------------------- |
| `-l`       | **Long format:** Shows file permissions, owner, size, and last modified date.            |
| `-a`       | **All:** Shows hidden files (those starting with a `.`).                                 |
| `-h`       | **Human-readable:** Used with `-l` to show file sizes in KB, MB, or GB instead of bytes. |
| `-R`       | **Recursive:** Lists files in all subdirectories as well.                                |
| `-t`       | **Time:** Sorts files by modification time, newest first.                                |
| -S         | Sort by the size                                                                         |
| -d         | View directory only                                                                      |
| -v         | natural sort                                                                             |
| -r         | reverse order of sorting                                                                 |

That directory part can also allow for the windcards. 

```bash
ls -d */ %% See only folders %%
```

`*` can also be used to filter the reuslt for example to get `.jpg` files

```bash
ls -h *.txt
```

`touch` canbe used to raete empty file , `mkdir` is used to create a folder with specific path. mkdir will fail if intermediate folders do not exist. use `-p` flag to create the intermdeitate folders as well. `rmdir` is used to delete empty directories.

`rm` is used to remove files and folders. First we can remove file using simple rm `rm file`. To remove entire folder and files use `-r` recursive flag. `rm -r file.txt`.

mv is used to move a folder / files (basically any inode). While cp is used to copy file. `mv` is just pointer update while cp is copying on actual disk as well. `cp -r` is used to recursiveln copy the folders as well. 

It puts all the content of file to terminal. To look at the files content we use cat. 

```bash
cat file
```

Options

- `-n` - Displays the line numbers
- `-A` - shows invisible like tabs or end of line merkers.
- It can be used to combine multiple files into 1

```bash
cat a.txt b.txt c.txt > out.txt
```

While `less` allows you to scroll and is used ofr bigger files. When you run `less filename`, you enter an interactive mode.

| **Key**       | **Action**                                                              |
| ------------- | ----------------------------------------------------------------------- |
| **Space**     | Scroll down one page.                                                   |
| **b**         | Scroll back (up) one page.                                              |
| **g** / **G** | Jump to the very start / Jump to the very end.                          |
| **/pattern**  | Search for a word (e.g., `/error`). Press **n** to find the next match. |
| **q**         | **Quit.** (The most important key!)                                     |

`less` opens it instantly because it only renders what you can see on the screen.

`tail and head` are used when users only care about the end or begining of files.By defaut they show only the first / last 10 lines. 

```bash
head -n <number of lines> file.txt
```

Example - 
```bash
tail -n +2 
```

+2 means to strip of first two lines. 

Also `tail -f file` is used to see the real time logs from a file. 

