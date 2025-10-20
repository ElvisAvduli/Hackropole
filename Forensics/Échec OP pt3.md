# Échec OP 3/3

## Challenge

The administrator tried to hide one of his IP addresses used to connect to this server. Recover the IP address and submit it as the flag in the format `FCSC{<ip-address>}`.

## Quick answer

```
FCSC{192.168.37.1}
```

## Summary of approach

1. Inspect the disk image and identify partitions.
2. Attach the image to a loop device and locate the encrypted partition.
3. Open the LUKS container using the known passphrase and expose the decrypted device.
4. Search raw device data (not just mounted filesystem) for artifacts from deleted logs — strings left in unallocated space often contain useful log fragments.
5. Correlate recovered log lines and fail2ban entries to identify the administrator IP.

## Disk & environment setup (commands)

```bash
# Attach the raw image to a loop device and create partition nodes
sudo losetup -P /dev/loop0 fcsc.raw

# Verify partitions
ls -l /dev/loop0*

# Identify filesystem types (shows LUKS on partition 3)
sudo blkid /dev/loop0p*
```

**Key finding:** `/dev/loop0p3` is LUKS-encrypted (TYPE="crypto_LUKS").

## Decrypt the container and examine devices

```bash
# Open the LUKS container (known passphrase: fcsc2022)
sudo cryptsetup luksOpen /dev/loop0p3 rootfs

# Confirm device mapping
lsblk -o NAME,TYPE,SIZE,MOUNTPOINT
```

This produces a decrypted block device at `/dev/mapper/rootfs` (or similar). Mounting the filesystem is useful for normal file access, but deleted log entries may only be recoverable by scanning the raw block device.

## Reasoning for raw device analysis

* The challenge hints that the administrator deleted logs. When files are removed, their filesystem metadata is updated but the underlying blocks can remain until reused. Forensic recovery often involves searching raw device contents for recognizable text patterns (timestamps, service names, IP addresses) rather than relying only on live logs.

## Evidence recovery (commands)

```bash
# Search for SSH accepted connections mentioning user 'obob'
sudo strings /dev/mapper/rootfs | grep "sshd.*Accepted.*obob" | head -20

# Search for fail2ban entries involving the recovered IP
sudo strings /dev/mapper/rootfs | grep "fail2ban.*192.168.37.1" || true
```

## Recovered log fragments (representative)

```
Mar 27 04:12:29 obob sshd[1571]: Accepted password for obob from 192.168.37.1 port 41864 ssh2
Mar 27 21:19:49 obob sshd[1308]: Accepted password for obob from 192.168.37.1 port 33028 ssh2
Mar 27 03:54:01 obob sshd[2161]: Accepted password for obob from 172.16.123.1 port 34520 ssh2
Mar 27 21:50:51 obob sshd[4956]: Accepted password for obob from 172.16.123.1 port 55266 ssh2

fail2ban.filter [5082]: INFO [sshd] Found 192.168.37.1 - 2022-03-27 04:12:26
```

**Interpretation:**

* `192.168.37.1` appears multiple times in SSH `Accepted` log fragments and is specifically flagged by fail2ban. These artifacts were found by scanning the raw decrypted device, indicating they were likely deleted from normal log files but remain in unallocated sectors.

## Why this IP is the flag

* Present in multiple SSH `Accepted` log fragments recovered from raw sectors.
* Appears in fail2ban log data indicating the security system explicitly detected activity from that IP.
* The administrator likely deleted or rotated logs to hide the connection, but raw device analysis recovered the evidence.

## Final flag

```
FCSC{192.168.37.1}
```

*(End of writeup)*
