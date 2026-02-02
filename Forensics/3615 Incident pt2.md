# 3615 Incident Part 2

## Objective

The context is identical to Part 1: a Windows system has been compromised by a ransomware, and paying the ransom is not an option. We are provided with the same memory image (`mem.dmp.tar.xz`).

This time, the goal is to **recover the encryption key** used by the ransomware.

Expected flag format:

```
ECSC{hey.hex()}
```

In other words, the flag is the hexadecimal representation of the encryption key.

---

## Methodology Overview

From Part 1, we already identified and extracted the ransomware (`assistance.exe`) and discovered a public repository containing its source code. Instead of reversing the binary, we can analyze the source to understand:

1. How the encryption key is generated.
2. How it is transmitted to the attacker.
3. What observable artifacts this process may leave in memory.

Once we have a precise model of the data exchanged by the malware, we can search for this pattern directly inside the memory dump and extract the key.

---

## 1. Source Code Analysis

### 1.1 Key Generation Logic

In `cmd/ransomware/ransomware.go`, the function `encryptFiles()` is responsible for orchestrating the encryption process. The relevant part is the following:

```go
// Generate the id and encryption key
keys["id"], _ = utils.GenerateRandomANString(32)
keys["enckey"], _ = utils.GenerateRandomANString(32)

// Persist the key pair on server
res, err := Client.AddNewKeyPair(keys["id"], keys["enckey"])
```

Two values are generated:

* `id`: a victim identifier
* `enckey`: the encryption key

Both are produced by `utils.GenerateRandomANString(32)`.

Looking at `utils/utils.go`:

```go
func GenerateRandomANString(size int) (string, error) {
    key := make([]byte, size)
    _, err := rand.Read(key)
    if err != nil {
        return "", err
    }

    return hex.EncodeToString(key)[:size], nil
}
```

This function:

1. Generates `size` random bytes.
2. Encodes them as hexadecimal.
3. Truncates the result to `size` characters.

Therefore, both `id` and `enckey` are **32-character hexadecimal strings**.

### 1.2 Transmission to the C2 Server

In `client/client.go`, the function `AddNewKeyPair()` shows how these values are sent:

```go
func (c *Client) AddNewKeyPair(id, encKey string) (*http.Response, error) {
    payload := fmt.Sprintf(`{"id": "%s", "enckey": "%s"}`, id, encKey)
    return c.SendEncryptedPayload("/api/keys/add", payload, map[string]string{})
}
```

So, before being encrypted and transmitted, the malware builds a JSON payload of the form:

```json
{
  "id": "<32-hex-chars>",
  "enckey": "<32-hex-chars>"
}
```

This is a crucial observation: even if the network traffic is encrypted, **this JSON string must exist in memory at some point**.

### 1.3 Summary of What We Are Looking For

From the code, we know that:

* The identifier `id` is a 32-character hexadecimal string.
* The encryption key `enckey` is also a 32-character hexadecimal string.
* At some point, both appear together in memory in a JSON structure:

```json
{"id": "<32 hex>", "enckey": "<32 hex>"}
```

This gives us a precise search pattern for the memory dump.

---

## 2. Searching the Memory Dump

### 2.1 Pattern-Based Search

Since we are looking for a textual JSON structure, a simple and effective approach is to use `strings` combined with `grep` and a regular expression.

We search for the following pattern:

* The literal string: `{ "id": "` followed by
* Exactly 32 word characters (`\w{32}`), then
* `", "enckey":`

Command used:

```bash
strings mem.dmp | grep -B 3 -A 3 -wE '{"id": "\w{32}", "enckey":'
```

This returns the following relevant excerpt:

```
{"id": "cd18c00bb476764220d05121867d62de", "enckey": "
cd18c00bb476764220d05121867d62de64e0821c53c7d161099be2188b6cac24
cd18c00bb476764220d05121867d62de64e0821c53c7d161099be2188b6cac24
95511870061fb3a2899aa6b2dc9838aa
422d81e7e1c2aa46aa51405c13fed15b
95511870061fb3a2899aa6b2dc9838aa
422d81e7e1c2aa46aa51405c13fed15b
```

We can clearly identify:

* `id = cd18c00bb476764220d05121867d62de`
* A long hexadecimal string following `"enckey":`, much longer than 32 characters.

This suggests that multiple values are present contiguously in memory, likely due to buffering or repeated operations.

---

## 3. Isolating the Candidate Keys

We know from the source code that the real encryption key must be **exactly 32 hexadecimal characters**.

So the strategy is:

1. Take the long hexadecimal blob.
2. Split it into chunks of 32 characters.
3. Remove duplicates.

After saving the blob to a file (e.g., `key.txt`), we can do:

```bash
fold -b32 key.txt | sort | uniq
```

Result:

```
422d81e7e1c2aa46aa51405c13fed15b
64e0821c53c7d161099be2188b6cac24
95511870061fb3a2899aa6b2dc9838aa
cd18c00bb476764220d05121867d62de
```

We obtain four 32-character hexadecimal candidates.

One of them is identical to the `id`:

```
cd18c00bb476764220d05121867d62de
```

Since the code generates `id` and `enckey` independently, and we already know the value of `id`, this one can be discarded. We are left with three plausible candidates.

---

## 4. Identifying the Correct Key

In a real-world scenario, the next step would be to:

* Identify the encryption algorithm.
* Extract an encrypted file from memory or disk.
* Attempt decryption with each candidate key.

However, within the context of this challenge, the flag format directly accepts the key. Therefore, we can test each remaining candidate as a flag:

```
ECSC{422d81e7e1c2aa46aa51405c13fed15b}  -> invalid
ECSC{64e0821c53c7d161099be2188b6cac24}  -> invalid
ECSC{95511870061fb3a2899aa6b2dc9838aa}  -> valid
```

---

## 5. Final Result

The correct encryption key is:

```
95511870061fb3a2899aa6b2dc9838aa
```

Final flag:

```
ECSC{95511870061fb3a2899aa6b2dc9838aa}
```
