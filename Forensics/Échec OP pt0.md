# Échec OP 0/3

## Challenge summary

This challenge gave us a disk image file named `fcsc.raw`. The goal was to find the **unique identifier (UUID) of the partition table** for the disk and submit it wrapped in the `FCSC{}` format as the flag.

> **Goal:** Find the disk GUID and format the flag as `FCSC{<GUID>}`.

## Tools used

* `gdisk` (GPT fdisk) — used to inspect GPT partition tables
* Any Unix-like shell (example commands shown below)

## Steps taken

1. Inspect the disk image with `gdisk -l` to list partition table information:

```bash
$ gdisk -l fcsc.raw
```

2. Review the output for the `Disk identifier (GUID)` line — this is the partition table UUID we need.

## Key `gdisk` output 

```
GPT fdisk (gdisk) version 1.0.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.
Disk fcsc.raw: 20971520 sectors, 10.0 GiB
Sector size (logical): 512 bytes
Disk identifier (GUID): 60DA4A85-6F6F-4043-8A38-0AB83853E6DC
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 4029 sectors (2.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048            4095   1024.0 KiB  EF02  
   2            4096         1861631   907.0 MiB   8300  
   3         1861632        20969471   9.1 GiB     8300  
```

## Extraction / Explanation

* The `gdisk -l fcsc.raw` command lists the GPT metadata for the disk image.
* The line `Disk identifier (GUID): 60DA4A85-6F6F-4043-8A38-0AB83853E6DC` is the unique identifier for the partition table (the GUID).

To produce the flag, we simply wrap that GUID inside `FCSC{}`.

## Flag

```
FCSC{60DA4A85-6F6F-4043-8A38-0AB83853E6DC}
```
