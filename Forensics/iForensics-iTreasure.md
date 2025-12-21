# iForensics — iTreasure

## Description

As you pass through customs, your phone is temporarily seized and later returned. Suspicious, you send the device to **ANSSI’s CERT-FR** for forensic analysis. Analysts extract two key datasets:

* an **iPhone backup**
* a **sysdiagnose collection**

Before the phone was handed over, the owner managed to **send a treasure**.
Your task: **find that treasure**.

The flag format is:

```
FCSC{<string_visible_in_the_image>}
```

---

## 1 — Extract the archives

Two compressed archives are provided. Extract them:

```bash
tar -xf backup.tar.xz
tar -xf sysdiagnose_and_crashes.tar.xz
```

`tar -xf` extracts the contents while preserving structure.

---

## 2 — Investigate Manifest.db

iOS backups store file metadata inside **Manifest.db**, mapping
logical iOS paths → hashed backup filenames.

Install SQLite if needed:

```bash
sudo apt install sqlite3
```

Open the database:

```bash
sqlite3 Manifest.db
```

View structure:

```sql
.schema
```

The important table is **Files**, which lists stored files and their original paths.

---

## 3 — Search for messages & emails

Since the challenge mentions something being **sent**, we focus on communication data
(Emails and SMS / iMessage attachments).

Query interesting paths:

```sql
SELECT fileID, relativePath FROM Files
WHERE relativePath LIKE "%sms%"
   OR relativePath LIKE "%mail%";
```

Nothing useful appears in email data.

However, we find a **message attachment**, more specifically a `.HEIC` image
located in the **Attachments** directory.
One relevant entry looks like:

```
6f/6f4e34098e00a80fde876c8638fb1d685be2318b
```

This is the hashed backup filename that corresponds to the real attachment.

---

## 4 — Recover the attachment

Copy the file and rename it:

```bash
cp 6f/6f4e34098e00a80fde876c8638fb1d685be2318b treasure.heic
```

`HEIC` is Apple’s image format, so we need to convert it.

---

## 5 — Convert HEIC to JPG

Install HEIC tools:

```bash
sudo apt install libheif-examples
```

Convert:

```bash
heif-convert treasure.heic treasure.jpg
```

---

## 6 — View the treasure

Open the image:

```bash
xdg-open treasure.jpg
```

(or `open treasure.jpg` on macOS)

Inside the image we find the **flag text**.

---

## Final Flag

```
FCSC{<string_visible_in_the_image>}
```
