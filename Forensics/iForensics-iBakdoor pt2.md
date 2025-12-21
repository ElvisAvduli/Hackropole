# iForensics — iBackdoor 2/2

## Description

As you pass through customs, your phone is temporarily seized and later returned. Suspicious, you send the device to **ANSSI’s CERT‑FR** for forensic analysis. Analysts collect:

* an **iPhone backup**
* a **sysdiagnose** package

After identifying the compromised application, we now want to understand **how the attacker retrieved the legitimate application before tampering with it**.

You must determine:

* **Identifier of the application used to retrieve the legitimate app**
* **Path where the legitimate app was stored**
* **Uninstall date of the legitimate app (local time)**

Flag format:

```
FCSC{<application identifier>|<path>|<date>}
```

Example:

```
FCSC{com.example|/private/var/tmp/test.xyz|2025-01-01 01:00:00}
```

---

## 1 — Start searching in sysdiagnose

This challenge relies heavily on **sysdiagnose logs**, which contain detailed filesystem and installation traces. Since we already know that Signal was compromised, we begin by searching anything related to Signal inside sysdiagnose.

First, move into the sysdiagnose directory and search for references:

```bash
cd sysdiagnose_and_crashes
grep -R "Signal" -n . | less
```

A particularly useful artifact is a **filesystem listing**, which records files existing at collection time. It is found here:

```
private/var/mobile/Library/Logs/CrashReporter/FilesystemMeta-*/private_var-dev_disk1s2.fslisting
```

You can open it with:

```bash
less private/var/mobile/Library/Logs/CrashReporter/FilesystemMeta-*/private_var-dev_disk1s2.fslisting
```

Inside this file, we find very interesting entries:

```
0   96  -  0  1744033538 16877 501 501 /private/var/mobile/Library/TrollDecrypt/
0   96  -  0  1744033541 16877 501 501 /private/var/mobile/Library/TrollDecrypt/decrypted/
66605056 66603101 - 0 1744033541 33188 501 501 /private/var/mobile/Library/TrollDecrypt/decrypted/Signal_7.53_decrypted.ipa
```

Two important observations:

**A strange directory name appears:** `TrollDecrypt`
**A decrypted Signal IPA exists:**

```
/private/var/mobile/Library/TrollDecrypt/decrypted/Signal_7.53_decrypted.ipa
```

`TrollDecrypt` is a known iOS tool that allows extracting and decrypting an installed application to generate a new `.ipa`, enabling later modification or reinjection.

This strongly suggests that the attacker **dumped the legitimate Signal app**, modified it, and reinstalled a backdoored version.

---

## 2 — Confirm activity in installation logs

Next, we validate this hypothesis using **MobileInstallation logs**, which track install / uninstall operations.

Navigate to the logs directory:

```bash
cd private/var/mobile/Library/Logs/CrashReporter/DiagnosticLogs/sysdiagnose/*/logs/MobileInstallation/
```

Search for uninstall traces:

```bash
grep "Uninstalling identifier" mobile_installation.log.*
```

Relevant output:

```
Mon Apr  7 07:40:47 2025 ... Uninstalling identifier org.whispersystems.signal
Mon Apr  7 07:43:55 2025 ... Uninstalling identifier com.fiore.trolldecrypt
Mon Apr  7 07:45:14 2025 ... Uninstalling identifier com.apple.calculator
```

From here we learn:

* The attacker used **TrollDecrypt**:

```
com.fiore.trolldecrypt
```

* The decrypted IPA was stored at:

```
/private/var/mobile/Library/TrollDecrypt/decrypted/Signal_7.53_decrypted.ipa
```

* Signal was uninstalled on:

```
2025-04-07 07:40:47 (local time)
```

This uninstall event corresponds to the moment the attacker removed the legitimate application so that the modified version could later be deployed.

---

## Final Flag

```
FCSC{com.fiore.trolldecrypt|/private/var/mobile/Library/TrollDecrypt/decrypted/Signal_7.53_decrypted.ipa|2025-04-07 07:40:47}
```

---

This confirms that the attacker decrypted the legitimate Signal application using **TrollDecrypt**, stored the extracted IPA in the TrollDecrypt directory, and uninstalled the original app shortly before replacing it with the compromised version.

```
FCSC{com.fiore.trolldecrypt|/private/var/mobile/Library/TrollDecrypt/decrypted/Signal_7.53_decrypted.ipa|2025-04-07 07:40:47}
```

