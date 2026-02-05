# SOCrate 5/6 

## Challenge Recap

In June 2023, a critical operator was compromised. The attacker wanted to retrieve a sensitive file but lacked permissions, so they modified ACLs and then downloaded the file. From the provided Windows Event Logs (`.evtx`), we must recover:

* The **victim FQDN**
* The **absolute path of the targeted file**
* The **source IP address of the attacker**

Flag format:

```
FCSC{FQDN_VICTIME|CHEMIN_ABSOLU|IP_SOURCE}
```

---

## Methodology

Because `.evtx` files are binary and mostly UTF-16LE encoded, we cannot `grep` them directly. We first extract readable strings using:

```bash
strings -e l *.evtx
```

Then we filter for relevant artifacts.

---

## Step 1 — Find the File Path and Hostname

The statement tells us the attacker used **icacls** to modify permissions. So we search for that command:

```bash
strings -e l *.evtx | grep -i "icacls" -C 5
```

### Observations

In the output, we repeatedly see commands like:

```
icacls C:\Users\jeanne.dias\.ssh\vm1.pem /grant Administrators:F
```

This directly gives us the **targeted file path**:

```
C:\Users\jeanne.dias\.ssh\vm1.pem
```

In the same event blocks, we also see the machine name field:

```
WORKSTATION2.cipherpol.gouv
```

So we extract:

* **File path:** `C:\Users\jeanne.dias\.ssh\vm1.pem`
* **FQDN:** `WORKSTATION2.cipherpol.gouv`

---

## Step 2 — Find the Source IP Address

The logs indicate that the relevant activity is tied to **Logon ID `0x84c794`**. We now search for this Logon ID to find the corresponding successful logon event (Event ID 4624):

```bash
strings -e l *.evtx | grep -F "0x84c794" -C 20
```

### Observations

In the Event ID 4624 block, we find the field:

```
IpAddress
172.16.45.110
```

So the **attacker source IP** is:

```
172.16.45.110
```

---

## Final Results

* **FQDN:** `WORKSTATION2.cipherpol.gouv`
* **File:** `C:\Users\jeanne.dias\.ssh\vm1.pem`
* **IP:** `172.16.45.110`

---

## Flag

```
FCSC{WORKSTATION2.cipherpol.gouv|C:\Users\jeanne.dias\.ssh\vm1.pem|172.16.45.110}
```
