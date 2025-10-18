# Analyse Memoire 1/2

## Overview

We use **Volatility3** to analyze a memory dump and extract three items:

1. **Name of the user** that used the machine
2. **Name of the machine** (computer name)
3. **Non-local IPv4 address** of the machine

For installation Guide, you can click [here](https://github.com/volatilityfoundation/volatility3)

---

## Prerequisites

* Volatility3 installed and accessible as `vol` (or use `python -m volatility3` depending on install).
* The memory dump file available (examples use `/path/to/analyse-memoire.dmp`).

---

## 1 — Find the user

**Command**

```bash
vol -f /path/to/analyse-memoire.dmp windows.envars | grep -Ei 'USERNAME'
```

**What to look for**

* Environment variables contain `USERNAME` entries.
* Look for accounts tied to interactive processes (e.g., `powershell.exe`, `explorer.exe`, `OneDrive.exe`).

**Result**

* **User:** `userfcsc-10`

---

## 2 — Find the machine (COMPUTERNAME)

**Command**

```bash
vol -f /path/to/analyse-memoire.dmp windows.envars | grep -Ei 'COMPUTERNAME'
```

**What to look for**

* `COMPUTERNAME` environment variable contains the machine name.

**Result**

* **Machine (COMPUTERNAME):** `DESKTOP-JV996VQ`

---

## 3 — Find a non-local IPv4 address

**Command**

```bash
vol -f /path/to/analyse-memoire.dmp windows.netscan
```

**What to look for**

* Inspect the `LocalAddr` column for IP addresses that are **not** `127.0.0.1` or `0.0.0.0`.
* Prefer addresses bound to real adapters (e.g., `10.x.x.x`, `192.168.x.x`, `172.16.x.x`).

**Result**

* **Non-local IPv4:** `10.0.2.15`

---

## Notes & tips

* If `vol` is not in your `PATH`, run Volatility3 like:

```bash
python -m volatility3 -f /path/to/dump <plugin> ...
```

* Use `| grep -i` to quickly find env variables but review surrounding lines in case values wrap.

* Save `netscan` output to a file for easier inspection:

```bash
vol -f file.dmp windows.netscan > netscan.txt
less netscan.txt
```

* If multiple candidate IPs appear, choose the one tied to a real network adapter and associated with established connections.

---

## Flag assembly

**Template**

```
FCSC{<username>:<computername>:<ipv4>}
```

**With the findings above:**

```
FCSC{userfcsc-10:DESKTOP-JV996VQ:10.0.2.15}
```
