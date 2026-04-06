# Crack the Hash — TryHackMe

## What is this room?

Crack the Hash is a beginner-level TryHackMe challenge focused on hash identification and cracking.
The goal is to identify each hash type, choose the right tool, and recover the original password.

---

## Core concept: what is a hash?

A hash is a one-way function. You put text in, you get a fixed-length string out.
You can't reverse it — but you can attack it.

```
"password123" → [MD5] → "482c811da5d5b4bc6d497ffa98491e38"
```

**How cracking works:**

| Method | Description |
| --- | --- |
| Rainbow Table | Pre-computed hash → plaintext database. Fast, but limited to known passwords. |
| Dictionary attack | Hash every word in a wordlist (e.g. rockyou.txt) and compare. |
| Brute force | Try every possible combination. Slow, but thorough. |

**How to identify a hash:**

The length and prefix tell you the algorithm before you touch any tool.

| Length | Prefix | Algorithm |
| --- | --- | --- |
| 32 chars | none | MD5 or MD4 |
| 40 chars | none | SHA-1 |
| 64 chars | none | SHA-256 |
| 60 chars | `$2y$` or `$2b$` | bcrypt |
| 106 chars | `$6$` | SHA-512crypt |

Tools used: [CrackStation](https://crackstation.net), [hashes.com](https://hashes.com), John the Ripper, Hashcat

---

## Task 1-1: MD5

**Hash:** `48bb6e862e54f2a795ffc4e541caed4d`

**Identification:** 32 characters, no prefix → MD5

**Method:** CrackStation (Rainbow Table lookup)

**Answer:** `easy`

---

## Task 1-2: SHA-1

**Hash:** `CBFDAC6008F9CAB4083784CBD1874F76618D2A97`

**Identification:** 40 characters, uppercase → SHA-1

**Method:** CrackStation

**Answer:** `password123`

---

## Task 1-3: SHA-256

**Hash:** `1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032`

**Identification:** 64 characters → SHA-256

**Method:** CrackStation

**Answer:** `letmein`

---

## Task 1-4: bcrypt

**Hash:** `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`

**Identification:** Prefix `$2y$`, 60 characters → bcrypt

**Tricky:** Rainbow Tables don't work here. bcrypt has a built-in salt, so the same password produces a different hash every time. CrackStation returns nothing.

The `12` in `$2y$12$` is the **cost factor** — the algorithm runs 2¹² = 4096 times intentionally to slow down attacks.

**Method:** hashes.com with GPU support, or John the Ripper:

```bash
echo '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom' > hash.txt
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

**Answer:** `bleh`

---

## Task 1-5: MD4

**Hash:** `279412f945939ba78ce0758d3fd83daa`

**Identification:** 32 characters → could be MD5 or MD4. Cracking reveals it's MD4.

**Method:** Hashcat with mode `-m 900`:

```bash
hashcat -m 900 hash.txt /usr/share/wordlists/rockyou.txt
```

**Answer:** `Eternity22`

---

## Task 2-1: SHA-256

**Hash:** `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85`

**Identification:** 64 characters → SHA-256

**Method:** CrackStation

**Answer:** `paule`

---

## Task 2-2: NTLM

**Hash:** `1DFECA0C002AE40B8619ECF94819CC1B`

**Identification:** 32 characters, but CrackStation identifies it as NTLM (Windows password hash)

**Tricky:** Looks like MD5 by length alone — always confirm with a tool like hashid or hashes.com.

**Method:** Hashcat with mode `-m 1000`:

```bash
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt
```

**Answer:** `n63umy8lkf4i`

---

## Task 2-3: SHA-512crypt

**Hash:** `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`

**Identification:** Prefix `$6$` → SHA-512crypt (Linux `/etc/shadow` format)

**Tricky:** Has an explicit salt (`aReallyHardSalt`). Rainbow Tables are useless — the salt makes every hash unique even for the same password.

**Method:** Hashcat with mode `-m 1800`:

```bash
hashcat -m 1800 hash.txt /usr/share/wordlists/rockyou.txt
```

**Answer:** `waka99`

---

## Task 2-4: SHA-1 with salt

**Hash:** `e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme`

**Identification:** The `:tryhackme` suffix is the salt. The hash itself is SHA-1.

**Tricky:** The format is `hash:salt`. You need to tell Hashcat to include the salt in the computation, otherwise it cracks nothing.

**Method:** Hashcat with mode `-m 110` (SHA-1 + salt appended):

```bash
hashcat -m 110 hash.txt /usr/share/wordlists/rockyou.txt
```

**Answer:** `481616481616`

---

## Key learnings

| Concept | Takeaway |
| --- | --- |
| Hash length | First filter for algorithm identification |
| Prefix (`$2y$`, `$6$`) | Modern algorithms self-identify |
| Rainbow Tables | Only work on unsalted, common hashes (MD5, SHA-1) |
| Salt | Makes each hash unique — kills Rainbow Tables |
| Cost factor | bcrypt is slow on purpose — that's the defense |
| Tool selection | CrackStation first (seconds), then Hashcat if needed |
| Wordlist attacks | Only crack weak passwords — strong + bcrypt = inviable |

## Real-world relevance

These aren't just CTF tricks. Every time a database leaks, these are the hashes attackers try to crack.
A system using MD5 with no salt is catastrophically vulnerable.
A system using bcrypt with a strong password policy is practically uncrackable.
**The algorithm matters. The password matters. Both together is what security looks like.**
**Also, you coul 100% use hashes.com or crackstation.com if you know what you are looking for**
