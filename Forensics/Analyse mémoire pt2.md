# Analyse mémoire - Pour commencer (2/2)

## Challenge Description

The memory dump was captured while a user was working on a highly sensitive document. If the workstation was compromised, this document may have been stolen. The task was to identify:

* The **name of the document editing software (executable)**.
* The **name of the document being edited** (filename only, without the full path).

**Flag format:**

```
FCSC{<software name>:<document name>}
```

Example: `FCSC{calc.exe:My accounts 2025.txt}`

---

## Analysis Steps

### 1. Identifying Running Processes

To find processes that might be associated with document editing, we ran the following Volatility3 command:

```bash
vol -f /mnt/c/Users/cyrha/Desktop/demo/file.dmp windows.pslist
```

* `pslist` enumerates all running processes in the memory dump.
* From the output, we noticed a process named `office.exe`, which could potentially be the document editing software.

### 2. Checking Process Command Lines

To verify which process was actually opening the sensitive document, we examined the command line arguments of running processes:

```bash
vol -f /mnt/c/Users/cyrha/Desktop/demo/file.dmp windows.cmdline | grep -Ei 'soffice.exe'
```

* `windows.cmdline` reveals the full command line used to start each process.
* This command returned:

```
soffice.exe [SECRET-SF][TLP-RED]Plan FCSC 2026.odt
```
## Findings

* **Document editing software:** `soffice.exe`
* **Document name:** `[SECRET-SF][TLP-RED]Plan FCSC 2026.odt`

---

## Flag

Based on the findings and the required format, the flag is:

```
FCSC{soffice.exe:[SECRET-SF][TLP-RED]Plan FCSC 2026.odt}
```

