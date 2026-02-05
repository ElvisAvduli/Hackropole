# SOCrate 6/6 

## Challenge Recap

> The action described in question 5 is carried out using a well-known offensive tool. What is the name of this tool?
>
> **Flag format:** `FCSC{name}`

In Part 5, we established that the attacker modified permissions on a sensitive file and then exfiltrated it. We now need to identify **which tool** was used to remotely execute these commands on the Windows host.

---

## Step 1 — Extract Evidence from Windows Logs

Windows `.evtx` files are binary and encoded in UTF-16LE. To search them, we use `strings` with the `-e l` option:

```bash
strings -e l *.evtx | grep "icacls" -C 2
```

### Key Artifact Found

From the output, the following command line appears multiple times:

```
cmd.exe /Q /c icacls.exe C:\Users\jeanne.dias\.ssh\vm1.pem /grant Administrators:F 1> \\127.0.0.1\ADMIN$\__1686913023.817504 2>&1
```

We also observe similar commands for permission reset:

```
cmd.exe /Q /c icacls.exe C:\Users\jeanne.dias\.ssh\vm1.pem /reset 1> \\127.0.0.1\ADMIN$\__1686913527.042923 2>&1
```

---

## Step 2 — Analyze the Execution Pattern

This command contains several important forensic fingerprints:

### 1. Non-interactive execution

```
cmd.exe /Q /c <command>
```

* `/Q /c` means: run command quietly, then exit
* This is typical of **remote execution tools**, not an interactive user session

### 2. Output redirection to ADMIN$ (SMB)

```
1> \\127.0.0.1\ADMIN$\__<timestamp> 2>&1
```

* `ADMIN$` is a hidden administrative SMB share (maps to `C:\Windows`)
* Stdout and stderr are redirected to a temporary file
* The attacker can then **retrieve this file over SMB** to read the command output

This technique is not random — it is a **known operational pattern** used by specific offensive tools.

---

## Step 3 — Correlate with the Parent Process

From earlier Windows events (notably Event ID 4688), the parent process of `cmd.exe` is:

```
WmiPrvSE.exe
```

This means:

* The command execution method is **WMI (Windows Management Instrumentation)**
* The output retrieval method is **SMB via ADMIN$**

---

## Step 4 — Identify the Tool

The combination of:

* WMI-based remote execution (`WmiPrvSE.exe` spawning `cmd.exe`)
* Non-interactive command execution (`cmd.exe /Q /c`)
* Output redirection to `\\127.0.0.1\ADMIN$\__<temp>`

is a **well-known fingerprint** of the following tool:

> **wmiexec** from the Impacket toolkit

### Why wmiexec?

* Native WMI does **not** return command output
* `wmiexec` solves this by:

  * Executing commands via WMI
  * Redirecting output to a temporary file on `ADMIN$`
  * Fetching the file over SMB for the attacker

This exact redirection pattern is characteristic of **Impacket's wmiexec.py**.

---

## Final Answer

**Tool used:** `wmiexec`

**Flag:**

```
FCSC{wmiexec}
```
