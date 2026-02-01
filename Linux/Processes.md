
---

Linux follows process hierarchy meaning each process has to be created by some other process except systemd which is the first process to get started by the kernel. Systemd is special and does some special work. Other processes need to be created via `fork` system call. 

#### Creating a process

In linux we can create the processes by only one method `fork` sys call. Once fork is called kernel duplicates the process assigns it an `pid`. In the begining both processes have set of resources(memory etc). Basically at this point child and parent both share exactly same memory. However as soon as child performs some instruction in its own process only the specific pages of memory get copied and separated out. This fature is called `copy on write` meaning until update is performed by child or parent they share exactly same memory address, but if one proecess performs the write first those pages are copied and then write. 

`Fork` is a special function that is called once but it returns twice once in parent process and other time in child process. Once returned both child and parent will continue to execute from that point onwards. 
Fork returns `pid` of child process in parent process and `0` in child process. This is usefull as we can figure out in which process we are. 

```python
import os

pid = os.fork()

if pid==0:
	print("CHILD",os.getpid()) # child process
else:
	print("PARENT: created pid",pid)
```

`os.getpid()` and `os.getppid()` are used to get the process pid and its parent pid respectively. 

Now to have entirely different process running on the child. We can have `exec` system call. This call replaces the current running process with a new one. This replacement includes destruction of `heap` and other resources.

While `exec` is not a single call rather its family of functions. We add some flags in suffix to tell how they behave - 

|**Function**|**Argument Style**|**Uses PATH?**|**Custom Env?**|
|---|---|---|---|
|`os.execl()`|List|No|No|
|`os.execp()`|List|**Yes**|No|
|`os.execv()`|Vector (Array)|No|No|
|`os.execvp()`|Vector (Array)|**Yes**|No|
|`os.execvpe()`|Vector (Array)|**Yes**|**Yes**|

Here `p` means os will look for the executable in the `$PATH` variable. Similarly `l` means ares arepassed as list of string. Now one quirk is that args are provided as followed for `l`
First arg has to be the executable and second is `arg0` of standard programs which is usually again the executable. Then we can give dynamic number of arguments. 

```python
os.execlp("sh","sh")
os.execlp("python","python","main.py")
```

Now depending upon the run time either child can stop first. Then if will release all the resources but will have an entry in the process table. Such kind of process is called zombie process. To deal with zombie process we use `os.wait()` call it pauses the parent until one of child is working. Wait returns a tuple with two - 

- First is pid - Process id of finished child
- status - 16 bit bunmber having exit code. 

we can use `os.getexitcodefromstatus` to get the exit code from status. Once wait call is regustered `parent` gets the pid and entry is removed from the process table. However if wait is not called we can have the schenario of `orphan` process where parent exits before child. Any orphan process is immediatly adopted by the `systemd`. 

#### Signalling 

Signalling in linux is the most primitive way of communication and also to notify some event has happened. 

Most common signals are - 
```
SIGINT (2) - interupt from keyboard
SIGTERM(15) - termination signal(please stop signal)
SIGKILL(9) - force kill
SIGCHILD(17) - sent to parent when child is killed
```

In addition there are other `SIGUSR1,...` which are used for communication. Signalling is very simple in `python`. User just has to register for some signal with an handler and that's it.

To send the signal we use `os.kill(pid,signal_name)`

```python 
import os
import time
import signal
import sys

def child_handler(signum, frame):
    if signum == signal.SIGUSR1:
        print(f"Child {os.getpid()}: I received a custom message (SIGUSR1)!")
    elif signum == signal.SIGTERM:
        print(f"Child {os.getpid()}: Received SIGTERM, exiting gracefully...")
        sys.exit(0)

pid = os.fork()

if pid == 0:
    signal.signal(signal.SIGUSR1, child_handler)
    signal.signal(signal.SIGTERM, child_handler)
    
    while True:
        print("Child is waiting for signals...")
        time.sleep(1)
else:
    print(f"Parent: Created child with PID {pid}")
    time.sleep(3)
    
    print(f"Parent: Sending SIGUSR1 to child {pid}")
    os.kill(pid, signal.SIGUSR1)
    
    time.sleep(2)
    print(f"Parent: Sending SIGTERM to child {pid}")
    os.kill(pid, signal.SIGTERM)
    
    finished_pid, status = os.wait()
    print(f"Parent: Child {finished_pid} has been reaped.")
```

#### Pipes

Better method of communication is using pipes. Pipe system call returns two file descriptors `read` and `write`. Pipe is one way channel and is treated like any other file descrptor. So main calls are - 

```cpp

(read,write) = os.pipe()

os.read()
os.write()

os.close(read/write)

```

Once pipe is created pipe should be made one way by explicit `close()`calls on one end `read` and on other `write`. If this is not done os will think its writing to itself and child hangs forever. Now the string before sending should be encoded. In Python 3, strings are Unicode (objects). However, the `os.write()` system call works at the kernel level, which only accepts **raw bytes**. Encoding converts that high-level string into a specific byte format. 

When you read from a pipe, you get a `bytes` object (e.g., `b'Hello'`). To use it as a standard string in your Python logic, you must decode it back into a Unicode string.

Note that encode and decode are string functions for format changing . 

os.read(fs,n) where n is number of bytes to be read is blocking call by default. If the pipe has data, kernel will return `n` bytes immediately. However if not available and writers are alive then it blocks. However if writer is not alive kernel returns empty bytes object. This is the signal that the stream is finished (EOF).

When you call `os.read(r, 1024)`, the `1024` is the **buffer size**.
- It tells the kernel: "Give me whatever you have, but no more than 1024 bytes."
- If there are only 5 bytes in the pipe, `read()` will return those 5 bytes immediately; it **does not** wait for the full 1024.

```python
import os

# os.pipe returns two file descriptors: r (read), w (write)
r, w = os.pipe()

pid = os.fork()

if pid > 0:
    # Parent: We want to write to the child, so close the read end
    os.close(r)
    message = "Hello from the Parent process!".encode()
    os.write(w, message)
    os.close(w)
    os.wait()
else:
    # Child: We want to read from the parent, so close the write end
    os.close(w)
    # Read up to 100 bytes
    data = os.read(r, 100)
    print(f"Child received: {data.decode()}")
    os.close(r)
    os._exit(0)
```

#### Rediection 

In linux every outside resouce can be accessed using file descriptor. Initially there are three file descriptors - 

0 - Standard input(stdin) - keyboard
1 - Standard outpu(stdout) - Screen
2 - Standard error(stderr) - Screen 

The dunction `os.dup2(old_fd,new_fd)`It takes the resource pointed to by `old_fd` and forcibly stamps it into the `new_fd` slot. If `new_fd` was already open (like your screen), the kernel silently closes it first.

```python
import os

# 1. Open a file. This will get the next available FD (likely 3)
file_fd = os.open("output.txt", os.O_WRONLY | os.O_CREAT | os.O_TRUNC)

print("This prints to the screen.")

# 2. THE TRICK: Duplicate our file FD into the STDOUT slot (1)
# After this line, anything sent to FD 1 goes to our file.
os.dup2(file_fd, 1)

# 3. We can now close the original file_fd because FD 1 is also pointing to it
os.close(file_fd)

# 4. Now, even though we use a standard print, it goes to the file!
print("This prints to output.txt!")

# 5. This also works for external programs via exec
os.execlp("echo", "echo", "I am echoing into the file from another program")
```

If you use `os.dup2(file_fd, 1)` and then call `print()`, Python's internal buffer might still try to send data to where the screen _used_ to be. In real system programming, we often follow this with a `sys.stdout.flush()` to ensure the data moves correctly.

`os.open()` is  direct wrapper to posix system call. It does not returns file object but rather an integer called file descriptor. 

|**Flag**|**Meaning**|**What it does**|
|---|---|---|
|**`os.O_WRONLY`**|**Write Only**|Opens the file for writing only. You cannot `read()` from this FD.|
|**`os.O_CREAT`**|**Create**|If the file doesn't exist, create it. Without this, the call fails if the file is missing.|
|**`os.O_TRUNC`**|**Truncate**|If the file _already_ exists, "truncate" its length to zero (delete all contents) as soon as it's opened.|
All of these are one bit flags and we use `|` or to combine them. 

Another example

```python
import os

r,w = os.pipe()
pid = os.fork()


if pid==0:
    os.close(r)
    os.dup2(w,1)
    os.execlp("ls","ls","-alhS")
    os.close(w)
else:
    os.close(w)
    data = os.read(r,1024)
    print("PARENT::",data)
    os.wait()
```

Piping data is heavy and slow as it has to go through kernel. 
### Shared memory and semaphores

Another way to talk is using shared memory. Since we can not allow raed and write at the same time we use semaphores to read and write one by one. 

Normally, the Operating System protects processes from each other. Process A cannot touch Process B's memory—if it tries, you get a `Segmentation Fault`.

**Shared Memory** is a special exception where the kernel maps the same physical RAM to the address space of multiple processes. The benefit is zero copying. 

Now to implement it we need shared memory using low level instruction and the locking is implemented using multiprocessing locking. 

```python
import os
import mmap
import time
from multiprocessing import Lock

# 1. Create Shared Memory (for the counter)
# We only need 4 bytes to store a standard integer
shared_mem = mmap.mmap(-1, 4, mmap.MAP_SHARED | mmap.PROT_WRITE | mmap.PROT_READ)
shared_mem.seek(0)
shared_mem.write(int(0).to_bytes(4, 'big')) # big endian

# 2. Create the Lock
# This MUST be created before the fork so both processes inherit the same lock
lock = Lock()

def increment_counter(process_name):
    for _ in range(5):
        # --- THE CRITICAL SECTION ---
        with lock:
            # Read current value
            shared_mem.seek(0)
            current_val = int.from_bytes(shared_mem.read(4), 'big')
            
            new_val = current_val + 1
            
            # Simulate a delay to make a race condition likely if lock fails
            time.sleep(0.1) 
            
            # Write back
            shared_mem.seek(0)
            shared_mem.write(new_val.to_bytes(4, 'big'))
            print(f"[{process_name}] Incremented to {new_val}")
        # --- LOCK RELEASED ---

pid = os.fork()

if pid == 0:
    # --- CHILD ---
    increment_counter("Child")
    os._exit(0)
else:
    # --- PARENT ---
    increment_counter("Parent")
    os.wait() # Wait for child to finish

    # Read final result
    shared_mem.seek(0)
    final_val = int.from_bytes(shared_mem.read(4), 'big')
    print(f"\nFinal Shared Counter: {final_val}")
    shared_mem.close()
```

#### Sockets 

A socket is an endpoint for communication. While pipes require a parent-child relationship, sockets allow **any** two processes to talk, whether they are on the same machine or on opposite sides of the planet.

To make a connection, the OS needs five pieces of information (the "5-tuple"):

- **Protocol:** (e.g., TCP or UDP)
- **Source IP:** Where is it coming from?
- **Source Port:** Which specific "door" is the sender using?
- **Destination IP:** Where is it going?
- **Destination Port:** Which "door" is the receiver listening on?

These sockets are of two types - 

- Stream socket(TCP)
- Datagram sockets (UDP)

A socket lifecycle is as follows - 

The Server side:
1. **`socket()`**: Create the socket.
2. **`bind()`**: Assign the socket to a specific address and port (e.g., `127.0.0.1:8080`).
3. **`listen()`**: Wait for someone to try to connect.
4. **`accept()`**: "Pick up the phone." This creates a **new** socket specifically for that caller.
5. **`recv()` / `send()`**: Exchange data.

The Client side:
1. **`socket()`**: Create the socket.
2. **`connect()`**: Dial the server's address and port.
3. **`send()` / `recv()`**: Exchange data.

Server

```python
import socket

# 1. Create a socket (AF_INET = IPv4, SOCK_STREAM = TCP)
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Bind to an address and port
server_socket.bind(('127.0.0.1', 65432))

# 3. Start listening
server_socket.listen()
print("Server is listening on 127.0.0.1:65432...")

# 4. Accept a connection (This blocks until someone connects)
conn, addr = server_socket.accept()
with conn:
    print(f"Connected by {addr}")
    while True:
        data = conn.recv(1024) # Receive up to 1024 bytes
        if not data:
            break
        print(f"Received: {data.decode()}")
        conn.sendall(b"Message Received!") # Echo back
```

Client

```python
import socket

# 1. Create the socket
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2. Connect to the server
client_socket.connect(('127.0.0.1', 65432))

# 3. Send data
client_socket.sendall(b"Hello from the Client!")

# 4. Receive response
data = client_socket.recv(1024)
print(f"Server said: {data.decode()}")

client_socket.close()
```

#### Multiprocessing module

The `multiprocessing` module is Python's high-level solution to the **Global Interpreter Lock (GIL)**. While threads in Python are limited to one CPU core at a time for execution, `multiprocessing` side-steps this by creating entirely separate memory spaces (separate instances of the Python interpreter).

It effectively automates the `fork`, `pipe`, `signal`, and `wait` logic we've been discussing manually.

We define a function we want to call as part of multiprocessing module and call it. 
Process takes two things  -  a function and a tuple of args however this just created process object to actually run it we have to `start` it. And finally `join` is higher level equivalent of `wait`. 

`start` 

```python
from multiprocessing import Process
import os

def info(title):
    print(f"{title} | Process ID: {os.getpid()}")

if __name__ == '__main__':
    p = Process(target=info, args=('Child Worker',))
    p.start() # This calls fork() under the hood
    p.join()  # This calls wait() under the hood
```

To run some other binary we usually use `subprocess.run()`.

```python
import multiprocessing
import subprocess

def run_external_binary(binary_path, args):
    print(f"Starting binary: {binary_path}")
    # This executes the binary and waits for it to finish
    result = subprocess.run([binary_path] + args, capture_output=True, text=True)
    print(f"Output from binary:\n{result.stdout}")

if __name__ == "__main__":
    # Define the binary and arguments
    # Example: running 'ping' on a website
    p = multiprocessing.Process(target=run_external_binary, args=("/sbin/ping", ["-c", "4", "google.com"]))
    
    p.start()
    print("Main script is still running while the binary works...")
    p.join()
    print("Binary execution complete.")
```

