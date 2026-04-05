# Bandit — OverTheWire

## What is Bandit?
Bandit is a wargame from OverTheWire designed to teach Linux fundamentals
through real challenges on an actual Linux server accessed via SSH.

---

## Level 0 → 1
**Goal:** Log into the server via SSH and find the password in a file called `readme`.

**Commands used:**
`ssh bandit0@bandit.labs.overthewire.org -p 2220`
`ls`
`cat readme`

---

## Level 1 → 2
**Goal:** Find the password in a file called `-`.

**Tricky:** `-` is interpreted as an argument, not a filename.

**Solution:**
`cat ./-`  or  `cat < ./-`

---

## Level 2 → 3
**Goal:** Find the password in a file with spaces in the filename.

**Tricky:** Spaces separate arguments in the terminal.

**Solution:**
`cat "spaces in this filename"`

---

## Level 3 → 4
**Goal:** Find a hidden file inside a directory.

**Tricky:** Hidden files start with `.` and don't show with regular `ls`.

**Solution:**
`ls -la`
`cat .hiddenfile`

---

## Level 4 → 5
**Goal:** Find the only human-readable file among many.

**Solution:**
`file ./*`  — shows file type of everything in the directory
`cat ./filename`  — open the one that says ASCII text

---

## Level 5 → 6
**Goal:** Find a file that is human-readable, 1033 bytes, not executable.

**Solution:**
`find . -type f -size 1033c ! -executable`
`cat ./maybehere07/.file2`

---
## The `find` command in depth

The `find` command searches for files using filters. You can combine
as many filters as you need in a single command.

### `-type` filter
Specifies what kind of thing you're looking for:

| Flag | Meaning | Example |
|------|---------|---------|
| `-type f` | files only | `find . -type f` |
| `-type d` | directories only | `find . -type d` |
| `-type l` | symbolic links only | `find . -type l` |

### `-size` filter
Searches by file size. The letter after the number sets the unit:

| Flag | Meaning | Example |
|------|---------|---------|
| `-size 1033c` | exactly 1033 bytes | `find . -size 1033c` |
| `-size +1000c` | bigger than 1000 bytes | `find . -size +1000c` |
| `-size -1000c` | smaller than 1000 bytes | `find . -size -1000c` |

### `!` — the NOT operator
Excludes results that match a condition.
Think of it as "everything EXCEPT this":
```bash
find . ! -executable        # files that are NOT executable
find . ! -type d            # everything that is NOT a directory
find . ! -name "*.txt"      # files that are NOT .txt
```

### Combining filters
Filters stack together, all conditions must be true at the same time:
```bash
# Find files that are:
# - actual files (not directories)
# - exactly 1033 bytes
# - not executable
find . -type f -size 1033c ! -executable
```

---

## Key learnings

| Trick | Solution |
|-------|----------|
| File named `-` | `cat ./-` |
| File with spaces | `cat "filename with spaces"` |
| Hidden files | `ls -la` |
| Find by size/type | `find . -type f -size 1033c ! -executable` |
| File type check | `file ./*` |


-------------------


## Level 6 → 7

**Goal:** Find the password stored somewhere on the server, owned by user bandit7, group bandit6, and exactly 33 bytes in size.

**Tricky:** The `find` command returns a lot of "Permission denied" errors that flood the output, making it hard to see the actual result. You can redirect stderr to clean up the output.

**Solution:**
```bash
find / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /path/to/file
```

**Stderr redirection:**
| Command | What it does |
|---|---|
| `2>/dev/null` | hides all errors, shows only results |
| `1>/dev/null` | hides results, shows only errors |
| `2>&1` | merges errors into the normal output |

---

## Level 7 → 8

**Goal:** Find the password in `data.txt` next to the word "millionth".

**Solution:**
```bash
grep "millionth" data.txt
```

---

## Level 8 → 9

**Goal:** Find the only line of text that appears once in `data.txt`.

**Tricky:** `uniq` only detects consecutive duplicates, so you need to sort first to group equal lines together.

**Solution:**
```bash
sort data.txt | uniq -u
```

| Flag | Meaning |
|---|---|
| `-u` | shows only lines that appear exactly once |
| `-d` | shows only duplicated lines |
| `-c` | counts occurrences of each line |

---

## Level 9 → 10

**Goal:** Find the password in `data.txt` among human-readable strings, preceded by several `=` characters.

**Solution:**
```bash
strings data.txt | grep "="
```

---

## Level 10 → 11

**Goal:** The password is stored in `data.txt`, encoded in base64.

**Solution:**
```bash
base64 -d data.txt
```

| Flag | Meaning |
|---|---|
| `-d` | decode (default encodes) |

---

## Level 11 → 12

**Goal:** The password is stored in `data.txt`, where all letters have been rotated 13 positions (ROT13). Case-sensitive.

**Tricky:** `tr` maps characters positionally, so uppercase and lowercase must be handled separately to preserve case.

**Solution:**
```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**How ROT13 works:**
ROT13 splits the alphabet in two halves of 13 and swaps them:
Input:   A B C D E F G H I J K L M | N O P Q R S T U V W X Y Z
Output:  N O P Q R S T U V W X Y Z | A B C D E F G H I J K L M

Applying ROT13 twice returns the original text, since the alphabet has 26 letters.

---

# Bandit — OverTheWire

## Level 12 → 13

**Goal:** The password for the next level is stored in a file called `data.txt` which is a hexdump (hexadecimal representation of binary data) that has been compressed multiple times.

**Understanding the challenge:**
- A hexdump is binary data represented as hexadecimal numbers
- The file has been compressed in multiple layers (gzip, bzip2, compress, etc.)
- You need to reverse both: convert hex to binary AND decompress multiple times

**Step-by-step solution:**

### Step 1: Create a temporary directory (good practice)
```bash
mkdir /tmp/bandit12
cd /tmp/bandit12
```

### Step 2: Copy the data file to your temporary directory
```bash
cp ~/data.txt .
```

### Step 3: Convert hexdump to binary
The file `data.txt` contains hexadecimal text. Use `xxd -r` to reverse it (convert hex → binary):
```bash
xxd -r data.txt data.bin
```

`xxd -r` means "reverse": converts hexadecimal representation back into actual binary data.

### Step 4: Check what type of file it is
```bash
file data.bin
```
Output: `gzip compressed data`

### Step 5: Decompress (loop until you get readable text)
Since there are multiple layers, you need to decompress several times. Use `-c` to output without modifying the original:
```bash
gunzip -c data.bin > data1
file data1
# Output: bzip2 compressed data

bunzip2 -c data1 > data2
file data2
# Output: gzip compressed data

gunzip -c data2 > data3
file data3
# Output: POSIX tar archive

tar -xf data3
ls
# Output: data5.bin

file data5.bin
# Output: bzip2 compressed data

bunzip2 -c data5.bin > data6
file data6
# Output: gzip compressed data

gunzip -c data6 > data7
file data7
# Output: POSIX tar archive

tar -xf data7
ls
# Output: data8.bin

file data8.bin
# Output: gzip compressed data

gunzip -c data8.bin > data9
file data9
# Output: ASCII text
```

### Step 6: Read the password
```bash
cat data9
```

**Key commands learned:**

| Command | What it does |
| --- | --- |
| `xxd -r` | Converts hexadecimal to binary |
| `gunzip -c` | Decompresses gzip without modifying original |
| `bunzip2 -c` | Decompresses bzip2 without modifying original |
| `tar -xf` | Extracts (unpacks) a tar archive |
| `file` | Identifies file type |

**Important note:**
Multiple compression types exist (gzip, bzip2, compress) because they were created at different times and for different purposes. In practice, you don't memorize which is "best" — you just use `file` to identify what type you have, then use the correct decompression command.

---

## Level 13 → 14

**Goal:** The password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user `bandit14`. You're given a private SSH key (`sshkey.private`) to authenticate as `bandit14`.

**The challenge:**
This level teaches you about SSH key-based authentication. Unlike the previous levels where you logged in with a password, you now need to use a private SSH key to authenticate as a different user.

**Why this is tricky:**
When you first try SSH from WITHIN the Bandit server, you get an error about localhost being blocked. This is the gotcha — OverTheWire deliberately blocks SSH connections from localhost to prevent resource abuse. **You must log in from your own machine, not from within the Bandit server.**

**Personal Note:**
I despise this level, whoever created OverTheWire, i'm going to find you and give you a hug.

---

## Step-by-step solution:

### Step 1: Download the SSH key to your local machine

You need to be on YOUR computer, not logged into Bandit.

From your local terminal (Windows Command Prompt, Mac Terminal, Linux Terminal):
```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:/home/bandit13/sshkey.private ~/
```

This command:
- `scp` = secure copy (like `cp` but over SSH)
- `-P 2220` = port 2220 (OverTheWire's port)
- `bandit13@bandit.labs.overthewire.org:/home/bandit13/sshkey.private` = source file on the server
- `~/` = destination on your machine (home directory)

When prompted, enter bandit13's password.

### Step 2: Use the SSH key to log in as bandit14

From your local machine:
```bash
ssh -i ~/sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

This command:
- `ssh` = secure shell (remote login)
- `-i ~/sshkey.private` = use THIS file as your identity/authentication key (instead of a password)
- `bandit14@bandit.labs.overthewire.org` = log in as user `bandit14` on this server
- `-p 2220` = use port 2220

### Step 3: Read the password
Once logged in as bandit14:
```bash
cat /etc/bandit_pass/bandit14
```

---

## Why I got stuck (and how to avoid it):

**The mistake:** Trying to run SSH from WITHIN the Bandit server
```bash
# DON'T DO THIS (from inside bandit13)
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
# Error: localhost connections blocked
```

**The solution:** Run SSH from your local machine
```bash
# DO THIS (from your computer's terminal)
ssh -i ~/sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
# Works!
```

**Why?** OverTheWire deliberately disables SSH connections from `127.0.0.1` (localhost) to prevent resource exhaustion. This means you can't chain SSH sessions. You must always connect directly from your machine to the server.

**Note:** OverTheWire previously allowed this, but changed the rules to conserve server resources. The level hint doesn't explicitly say this, which is why it's confusing.

---

## Key concepts:

| Concept | Explanation |
| --- | --- |
| **Private SSH key** | A file that proves your identity to a server (like a super-secure password) |
| **`-i` flag in SSH** | "identity file" — tells SSH which key to use for authentication |
| **`scp` command** | Secure copy — transfers files over SSH |
| **localhost blocking** | Bandit disables SSH from `127.0.0.1` to prevent abuse |
| **SSH from your machine** | Always connect directly from your computer, never try to chain SSH sessions |

---

## Summary

- **Bandit 12→13:** Learn about hexdumps, multiple compression layers, and how to decompress systematically
- **Bandit 13→14:** Learn about SSH key-based authentication and the importance of connecting from your local machine, not from the server itself
