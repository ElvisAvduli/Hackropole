# Small Notes

**Description:** *Do you know the PCAPng format well?*`
---

## Overview

We are given a `.pcapng` capture file and asked to find the hidden flag. The hint in the description suggests that the solution relies on **understanding the PCAPng format**, not just inspecting packet payloads.

Unlike classic `.pcap`, the **PCAPng** format supports additional metadata such as **packet comments**. These comments can be abused in CTF challenges to hide information in plain sight.

---

## 🔍 Step 1 — Open the Capture

Open the file with **Wireshark**:

```
File → Open → petites-notes.pcapng
```

At first glance, the traffic itself does not reveal anything interesting: no obvious plaintext flag appears in packet payloads.

---

## Step 2 — Remember PCAPng Features

PCAPng supports:

* Per-packet comments
* Capture comments
* Interface metadata

So instead of only looking at the packet data, we should also check **packet comments**.

In Wireshark, you can:

* Click on packets and look for a **Comments** field, or
* Enable the **Packet Comments** column

---

## Step 3 — Extract the Hidden Notes

Several packets contain short text fragments inside their **packet comments**. The extracted pieces are:

```
ECSC{cShl
e5dO
KYBfj
LNzT}
```

Each fragment alone looks incomplete, but together they clearly form a flag.

---

## Step 4 — Reassemble the Flag

Concatenating the fragments in order:

```
ECSC{cShl + e5dO + KYBfj + LNzT}
```

Gives:

```
ECSC{cShle5dOKYBfjLNzT}
```
