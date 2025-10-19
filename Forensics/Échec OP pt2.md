# Échec OP 2/3

## Challenge summary

This task continues from the previous disk analysis. While exploring the mounted filesystem, we discovered a `shadow`-style file and evidence that a user `obob` exists. By searching for occurrences of `obob` and inspecting command history, we recovered a secret string that becomes the flag.

**Goal:** Find the secret string and format the flag as `FCSC{<string>}`.

## Overview of approach

1. Inspect `/etc/shadow` (or a shadow-like file) to identify user accounts and spot any interesting usernames.
2. Search the filesystem for references to the interesting username (`obob`).
3. Inspect shell history files (root's and others') to find any plaintext secrets typed into the shell (e.g., during `passwd` usage).
4. Extract the secret and wrap it into `FCSC{}`.

## Commands used and why (step-by-step)

### 1) `sudo cat shadow` (or `sudo cat /etc/shadow` if mounted)

**Why:** The `shadow` file lists local user accounts and their hashed passwords (or markers like `*`, `!`, `!!`). It helps identify accounts of interest. In this case the file excerpt shows a user called `obob` with a password hash.

**Observed snippet:**

```
obob:$6$cvD51kQkFtMohr9Q$vE2L5CUX3jDZgVUZGOFNUFsSHGomH/EP5yYQA3dcKMm9U00mvA9pLzo7Z.Ki6exchu29jEENxtBdGUXCISNxL0:19078:0:99999:7:::
```

This confirms `obob` exists.

### 2) `sudo grep "obob" -r *`

**Why:** Recursively search the mounted filesystem for occurrences of the username; this often finds configuration files, home directories, or shell histories that mention the account.

**What we look for:** files like `*/.bash_history`, `/home/obob/*`, scripts, backups, and any references where the user appears.

**Example hit:**

```
root/.bash_history:passwd obob
```

This indicates the root user executed `passwd obob` at some point and that the command (and what followed) might be in root's bash history.

### 3) `sudo cat /mnt/root/.bash_history`

**Why:** Root's shell history can contain sensitive plaintext that was typed interactively, such as passwords entered to `passwd` if the administrator accidentally typed the password on the command line or the terminal echoed input into history. Inspecting `.bash_history` is faster than attempting to crack hashes in `shadow`.

**What we expect to see:** A sequence of commands; in this case we find evidence of `passwd obob` followed by what looks like a password on the next line.

**Observed excerpt:**

```
exit
passwd obob
CZSITvQm2MBT+n1nxgghCJ
exit
```

Interpretation: after `passwd obob` the next line in the history is the plaintext password that was entered (likely pasted/typed) — `CZSITvQm2MBT+n1nxgghCJ`.

### 4) Construct the flag

Per the challenge instructions, wrap the secret string into `FCSC{}`.

## Final flag

```
FCSC{CZSITvQm2MBT+n1nxgghCJ}
```
