# Échec OP 1/3
## Challenge summary

You are given a disk image `fcsc.raw` containing an encrypted disk. The administrator used LUKS encryption with a known passphrase. The challenge asks:

> **What is the date of creation of the filesystem in UTC?**

The flag must be in ISO 8601 format inside `FCSC{}`. Example: `FCSC{2022-04-22T06:59:59Z}`.

## Quick answer (flag)

```
FCSC{2022-03-27T03:44:49Z}
```

## Tools & commands used (and why)

Below are the exact commands used during the investigation and an explanation of *why* each command is important.

### 1) `sudo losetup -f --show -P fcsc.raw`

**What it does:**

* `losetup` attaches a file containing a block device image (here `fcsc.raw`) to a loop device (e.g., `/dev/loop0`).
* `-f --show` finds the next free loop device and prints its name.
* `-P` forces the kernel to scan for partitions and create loop device partition nodes (like `/dev/loop0p1`).

**Why we run it:**

* We need a block device we can pass to partitioning and LUKS/cryptsetup tools. Attaching `fcsc.raw` to `/dev/loop0` exposes its partitions as block devices.

**Example output:**

```
/dev/loop0
```

### 2) `sudo fdisk /dev/loop0` (then `p` to print)

**What it does:**

* `fdisk` inspects and manipulates partition tables. The `p` command prints partition layout.

**Why we run it:**

* To confirm partition layout, partition types, sizes, and the disk identifier. In particular, it shows which partition likely contains the LUKS/container we need to open.

**Relevant output (trimmed):**

```
Disk /dev/loop0: 10 GiB, ...
Disklabel type: gpt
Disk identifier: 60DA4A85-6F6F-4043-8A38-0AB83853E6DC

Device         Start      End  Sectors  Size Type
/dev/loop0p1    2048     4095     2048    1M BIOS boot
/dev/loop0p2    4096  1861631  1857536  907M Linux filesystem
/dev/loop0p3 1861632 20969471 19107840  9.1G Linux filesystem
```

**Interpretation:**

* Partition 3 (`/dev/loop0p3`) is the large 9.1 GiB partition — this is the likely encrypted LUKS partition.

### 3) `sudo cryptsetup luksOpen /dev/loop0p3 cryptpart` and passphrase `fcsc2022`

**What it does:**

* `cryptsetup luksOpen` opens a LUKS encrypted partition and maps it to a device mapper name (here `cryptpart`). It essentially decrypts the container (after verifying the passphrase) and provides an unlocked block device at `/dev/mapper/cryptpart`.

**Why we run it:**

* The filesystem we want (or the LVM metadata containing the filesystem) sits inside the encrypted partition. We must open it to access the inner structures.

### 4) `sudo mount /dev/ubuntu-vg/ubuntu-lv /mnt`

**What it does:**

* After unlocking the LUKS device, it contained LVM physical volumes and logical volumes. One logical volume was named `/dev/ubuntu-vg/ubuntu-lv`. Mounting that device at `/mnt` gives us access to the filesystem contents.

**Why we run it:**

* To examine files or confirm the filesystem is present and accessible. (For this specific task we don't need to inspect file contents — we only need the creation timestamp — but mounting verifies the logical volume is correct.)

### 5) `sudo lvdisplay`

**What it does:**

* Shows LVM logical volume metadata including LV path, name, size, and important: the LV Creation host and time.

**Why we run it:**

* The `lvdisplay` output includes the LV Creation time, which in practice often coincides with the time the logical volume (and usually the filesystem within) was created. This timestamp is our best forensic evidence for the filesystem creation time when explicit filesystem metadata isn't available or as a corroborating timestamp.

**Relevant output (trimmed):**

```
--- Logical volume ---
LV Path                /dev/ubuntu-vg/ubuntu-lv
LV Name                ubuntu-lv
VG Name                ubuntu-vg
LV UUID                W4Y1My-22pb-DbM1-o1IU-dBKO-pJ6O-FcE7sG
LV Write Access        read/write
LV Creation host, time ubuntu-server, 2022-03-26 23:44:49 -0400
LV Status              available
...
LV Size                9.09 GiB
```

## Determining the filesystem creation time (UTC)

* The `lvdisplay` shows `LV Creation host, time ubuntu-server, 2022-03-26 23:44:49 -0400`.
* That timestamp includes an explicit timezone offset of `-0400` (i.e., four hours behind UTC).

**Convert to UTC:**

* Add 4 hours to the local time to get UTC.

```
2022-03-26 23:44:49 -0400  =>  UTC = 2022-03-27 03:44:49Z
```

**Reasoning / assumptions:**

* The LVM logical volume creation time is recorded by LVM and is a reliable timestamp for when the LV (and likely the filesystem inside) was created.
* In some rare cases, a filesystem might be created at a slightly different time than the LV (e.g., if the LV was created and the filesystem immediately created a bit later). The challenge text and the captured evidence indicate the intended answer is the LV creation timestamp converted to UTC.

## Final flag

The final flag is the ISO 8601 UTC timestamp wrapped in `FCSC{}`:

```
FCSC{2022-03-27T03:44:49Z}
```
