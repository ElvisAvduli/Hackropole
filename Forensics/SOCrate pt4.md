# SOCrate 4/6 

**Scenario:**

> In June 2023, a vital operator was the victim of an attack that compromises its entire information system. You have received the Linux and Windows logs and you have to answer questions from the investigators.
>
> The tool identified in question 3 performed several LDAP requests. Find the **IP** and **FQDN** of the machine targeted by these queries.
>
> **Flag format:** `FCSC{IP|FQDN}`

---

## Objective

Identify the **internal host** targeted through the Chisel tunnel (reverse SOCKS) by correlating:

* **Linux AuditD logs** (to find the destination IP and service/port), and
* **Windows Event Logs** (to resolve that IP to a hostname/FQDN).

---

## Investigation Logic

From Part 3, we know the attacker deployed **Chisel** to create a **reverse SOCKS proxy**. This typically enables **lateral movement** toward directory services or file services. A common first target in Windows environments is **Active Directory over LDAP (port 389)**.

So the plan is:

1. On Linux, look for network activity from the `text` (Chisel) process toward **port 389**.
2. Extract the **destination IP**.
3. Pivot to Windows logs to map that IP to a **hostname/FQDN**.

---

## Step 1 — Find the Target IP (Linux Logs)

We filter AuditD logs for the `text` binary and then look for LDAP traffic (port **389**):

```bash
# Filter audit logs for the 'text' binary and look for LDAP traffic (port 389)
for f in *.log; do
  ausearch -i -if "$f" -c text 2>/dev/null;
done | grep "389"
```

**What this does:**

* Iterates over all `.log` files
* Decodes AuditD entries with `-i`
* Filters on executions related to the `text` binary (Chisel)
* Keeps only lines referencing port **389** (LDAP)

**Observed artifact (example):**

```
node=webserver ... saddr={ saddr_fam=inet laddr=172.16.42.10 lport=389 }
```

**Analysis:**

* The process `text` initiates connections to **172.16.42.10** on **port 389**.
* Port 389 corresponds to **LDAP**, strongly suggesting the attacker is querying **Active Directory**.

**Conclusion (Linux side):**

* **Target IP:** `172.16.42.10`

---

## Step 2 — Resolve the IP to a Hostname (Windows Logs)

Linux logs only give us the IP address. To find the **machine name**, we pivot to the Windows Event Logs (`.evtx`). These files are encoded in **UTF-16LE**, so we use `strings -e l` to extract readable text and search for the IP:

```bash
# Search Windows logs for the IP to find the associated hostname
strings -e l *.evtx | grep -C 5 "172.16.42.10"
```

**Result:**

The surrounding context shows a Windows authentication or directory-related event (e.g., Event ID 4624 / LDAP-related activity) where the IP **172.16.42.10** is associated with the server name:

```
DC01-SRV.cipherpol.gouv
```

**Analysis:**

* The IP **172.16.42.10** maps to **DC01-SRV.cipherpol.gouv**.
* The hostname strongly indicates a **Domain Controller** (DC01), which matches the LDAP targeting observed on Linux.

**Conclusion (Windows side):**

* **FQDN:** `DC01-SRV.cipherpol.gouv`

---

## Correlation & Attacker Intent

By combining both sides:

* **Linux** shows Chisel (`text`) connecting to **172.16.42.10:389**
* **Windows** confirms **172.16.42.10 = DC01-SRV.cipherpol.gouv**

This confirms the attacker used the SOCKS tunnel to **pivot toward the Domain Controller** and perform **LDAP queries**, a classic step toward **credential harvesting and domain compromise**.

---

## Final Answer

```
FCSC{172.16.42.10|DC01-SRV.cipherpol.gouv}
```
