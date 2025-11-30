# iForensics — iWifi

## Description

As you pass through customs, your phone is temporarily taken and returned later. Suspicious, you send the device to ANSSI’s CERT‑FR for forensic analysis. A `sysdiagnose` capture and a backup extraction were performed.

This challenge asks you to recover three pieces of information from the backup:

* **SSID** (Wi‑Fi network name)
* **BSSID** (MAC address of the access point)
* **iCloud account** associated with the phone

The flag format is:

```
FCSC{<SSID>|<BSSID>|<iCloud account>}
```

---

## 1 — Prepare the backup

Extract the backup archive:

```bash
tar -xf backup.tar.xz
cd backup
```

## 2 — Locate Wi‑Fi configuration files in Manifest.db

Use `Manifest.db` to avoid searching manually through thousands of hashed files.
Open the database:

```bash
sqlite3 Manifest.db
```

Search for Wi‑Fi‑related paths:

```sql
SELECT fileID, relativePath FROM Files WHERE relativePath LIKE "%wifi%";
```

One of the relevant entries is:

```
0f/0fa75546343ba224c9fe55adc73e8fdedc1029c3
```

## 3 — Convert Wi‑Fi plist to XML

The file is a **binary plist**. Convert it to XML for readability:

```bash
plistutil -f xml -i 0f/0fa75546343ba224c9fe55adc73e8fdedc1029c3 -o wifi.xml
```

On macOS:

```bash
plutil -convert xml1 0f/0fa75546343ba224c9fe55adc73e8fdedc1029c3 -o wifi.xml
```

## 4 — Extract SSID & BSSID

Example structure found inside `wifi.xml`:

```xml
<key>SSID</key>
<string>FCSC</string>

<key>BSSID</key>
<string>66:20:95:6c:9b:37</string>
```

## 5 — Extract the iCloud account

To locate the iCloud account, search through files for email strings:

```bash
grep -r ".com" .
```

Relevant match after trying:

```
35/3563f4a234af8c67a8a6a664d5e70fa131739c2f
```

Inside this plist:

```xml
<plist version="1.0">
  <dict>
    <key>registration.savedAccountName</key>
    <string>robertswigert@icloud.com</string>
  </dict>
</plist>
```

---

## Final Flag

```
FCSC{FCSC|66:20:95:6c:9b:37|robertswigert@icloud.com}
```
