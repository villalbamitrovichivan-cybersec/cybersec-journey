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
