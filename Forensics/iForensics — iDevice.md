# iForensics — iDevice

## Description

As you pass through customs, you're asked to hand over your phone along with its unlock code. When it is returned a few hours later, suspicion arises — so the device is sent to ANSSI’s CERT-FR for forensic analysis. A sysdiagnose and a backup extraction are performed.

Our first objective is to recover **the iOS version and device model identifier**, which will allow us to construct the flag.

The expected format is:

```
FCSC{<model identifier>|<build number>}
```

**Example:** iPhone 14 Pro Max running iOS 18.4 (Build 22E240) → `FCSC{iPhone15,3|22E240}`

---

## 1 — Extract the backup archive

We are provided with:

```
backup.tar.xz
```

Begin by extracting the archive:

```bash
tar -xf backup.tar.xz
```

Once decompressed, we browse through the extracted directory structure.

---

## 2 — Searching for system information

We are looking for **Build Version** and **Model Identifier**.

To search efficiently, we can use `grep`:

```bash
grep -r "Build Version" 
```

This reveals a relevant entry inside an `Info.plist` file.

Opening the file, we locate the values required for the flag:

```
<key>Build Version</key>
<string>20A362</string>
.
.
.
<key>Product Type</key>
<string>iPhone12,3</string>
```

---

## Flag

```
FCSC{iPhone12,3|20A362}
```
