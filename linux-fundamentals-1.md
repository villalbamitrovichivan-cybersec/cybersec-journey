# Linux Fundamentals 1 — TryHackMe

## What is Linux?
Linux is an operating system present in practically everywhere, 
from servers to embedded devices. It prioritizes efficiency 
over an elaborate visual interface.

Unlike Windows, Linux has multiple distributions (distros), 
each with its own specialty and purpose. For example:
- **Ubuntu** — aimed at general users, more beginner-friendly
- **Kali Linux** — designed specifically for cybersecurity
- **Debian** — the base of many other distros, very stable

A key advantage is that it is extremely lightweight, being able to run 
on systems with very limited RAM and processing resources.

## Basic Commands

| Command | Description | Example |
|---------|-------------|---------|
| `echo` | Prints text to the terminal | `echo "Hello World"` |
| `whoami` | Shows the current logged-in user | `whoami` |
| `ls` | Lists everything in the current directory | `ls -la` |
| `cd` | Changes directory | `cd /home` |
| `pwd` | Shows current location in the file system | `pwd` |
| `cat` | Reads or outputs file content | `cat file.txt` |
| `find` | Searches for files or directories | `find / -name file.txt` |
| `grep` | Searches for specific content inside a file | `grep "password" file.txt` |
| `>` | Creates a file or replaces its content | `echo "text" > file.txt` |
| `>>` | Appends content to an existing file | `echo "more" >> file.txt` |
| `&` | Runs a command in the background | `command &` |
| `&&` | Runs multiple commands in sequence | `cmd1 && cmd2` |

## What caught my attention
The simplicity of it all. It feels "old school" in the best way possible.
Coming from a lifetime of graphical interfaces, using only text commands 
feels unusual at first, but I can already see the potential — once you 
build the muscle memory, it's faster and more precise than any GUI.

## What was difficult
The lack of experience with the terminal made everything feel slow 
and unfamiliar. Every command had to be thought through consciously 
instead of being instinctive.

The main way to compensate this is practice — setting up a Kali Linux VM 
to test commands daily until it becomes second nature. The goal is to 
stop thinking about the syntax and just use it.
