# 3615 Incident Pt. 1

> One more victim fell to ransomware. Paying the ransom is not an option, so we are asked to investigate the system and recover information about the attack.
>
> The goal of this first part is to find:
>
> * The name of the ransomware executable
> * Its process identifier (PID)
> * The SHA1 of the filename `flag.docx` **after encryption**
>
> **Flag format:** `ECSC{ransomware.exe:pid:sha1}`
>
> We are given a memory dump: `mem.dmp.tar.xz`, which contains a snapshot of the system’s RAM at the time of acquisition.

---

## Tools

* **Volatility 3** – memory forensics framework
* **strings / grep** – quick triage
* **sha1sum / base64** – hashing and encoding
* **PEStudio** (optional) – static analysis of the extracted binary

---

## Identifying the Operating System

Before using Volatility, we need to know what OS the memory dump comes from.

A quick triage with `strings` already gives us a hint:

```bash
strings mem.dmp | grep "Windows"
```

We can see many references to *Microsoft Windows*, so this is very likely a Windows memory dump.

Let’s confirm this with Volatility:

```bash
python3 vol.py -f mem.dmp windows.info
```

Relevant output:

```
NtMajorVersion    10
Is64Bit           True
Major/Minor       15.10586
SystemTime        2019-05-08 20:04:11+00:00
```

So we are dealing with:

* **Windows 10**
* **64-bit architecture**
* **Build 10586**

Now we can safely use the Windows plugins.

---

## Process Analysis — Finding the Ransomware

To list running processes, we use:

```bash
python3 vol.py -f mem.dmp windows.pstree
```

Among many legitimate processes (`explorer.exe`, `firefox.exe`, `notepad.exe`, etc.), one stands out:

```
PID   PPID   ImageFileName     CreateTime
5208  3184   assistance.exe    2019-05-08 20:00:16
```

`assistance.exe` is **not** a standard Windows process and looks suspicious. This is very likely our ransomware.

So we already have:

* **Executable name:** `assistance.exe`
* **PID:** `5208`

Let’s extract it from memory.

---

## Extracting `assistance.exe` from the Memory Dump

First, we locate the file in memory:

```bash
python3 vol.py -f mem.dmp windows.filescan | grep "assistance.exe"
```

Example result:

```
0xe00011483b40  \Users\...\Downloads\assistance.exe
```

Now we dump it using that virtual address:

```bash
python3 vol.py -f mem.dmp windows.dumpfiles --virtaddr 0xe00011483b40
```

This gives us an extracted PE file:

```
file.0xe00011483b40.0xe0001219c830.ImageSectionObject.assistance.exe.img
```
---

## Quick Malware Triage

Running `strings` on the extracted binary reveals something interesting:

```
github.com/mauri870/ransomware
```

This points to a **public GitHub repository containing the ransomware source code**.

So this confirms:

* `assistance.exe` is indeed the ransomware
* We can analyze the source code to understand how files are renamed after encryption

At this stage, we have two parts of the flag:

```
ECSC{assistance.exe:5208:????}
```

Now we just need the SHA1 of the encrypted filename of `flag.docx`.

---

## Understanding How Files Are Renamed

Looking at the source code (`/cmd/ransomware/ransomware.go`), we find this function:

```go
func encryptFiles() {
    // Rename the files after all have been encrypted
    for _, file := range FilesToRename.Files {
        newpath := strings.Replace(
            file.Path,
            file.Name(),
            base64.StdEncoding.EncodeToString([]byte(file.Name())),
            -1,
        )
        err := utils.RenameFile(file.Path, newpath + cmd.EncryptionExtension)
        if err != nil {
            continue
        }
    }
}
```

And in `common.go`:

```go
EncryptionExtension = ".encrypted"
```

So the ransomware:

1. Takes the original filename
2. Encodes it in **Base64**
3. Appends an extension (in our dump, we observe `.chiffré` instead of `.encrypted`)

---

## Finding the Encrypted `flag.docx`

First, we Base64-encode `flag.docx`:

```bash
echo -n "flag.docx" | base64
```

Result:

```
ZmxhZy5kb2N4
```

Now we search for this in the memory dump:

```bash
python3 vol.py -f mem.dmp windows.filescan | grep "ZmxhZy5kb2N4"
```

We find:

```
\ZmxhZy5kb2N4.chiffré
```

So the encrypted filename is:

```
ZmxhZy5kb2N4.chiffré
```

---

## Computing the SHA1 of the Encrypted Filename

Now we compute the SHA1 of the **filename string** (not the file content):

```bash
echo -n "ZmxhZy5kb2N4.chiffré" | sha1sum
```

Result:

```
c9a12b109a58361ff1381fceccdcdcade3ec595a
```

---

## Final Flag

We now have all three elements:

* Ransomware name: `assistance.exe`
* PID: `5208`
* SHA1 of encrypted filename: `c9a12b109a58361ff1381fceccdcdcade3ec595a`

**Final flag:**

```
ECSC{assistance.exe:5208:c9a12b109a58361ff1381fceccdcdcade3ec595a}
```
