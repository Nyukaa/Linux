# Chapter 7. Manage Basic Storage

Goal: create and manage partitions, file systems and swap from the command line.

Sections:

1. Add Partitions, File Systems, and Persistent Mounts
2. Manage Swap Space

## 1. Add Partitions, File Systems, and Persistent Mounts

Flow: partition the disk, create a file system, mount it, add it to `/etc/fstab`.

|                | MBR                                        | GPT   |
| -------------- | ------------------------------------------ | ----- |
| Firmware       | BIOS                                       | UEFI  |
| Max partitions | 4 primary (up to 15 with extended/logical) | 128   |
| Max disk size  | 2 TiB                                      | 8 ZiB |

```bash
parted /dev/vdb mklabel gpt                         # write a label (wipes the old table!)
parted /dev/vdb mkpart userdata xfs 2048s 1000MB    # create a partition
parted /dev/vdb print                               # show partitions
parted /dev/vdb rm 1                                # delete partition 1
udevadm settle                                      # wait for /dev/vdb1
```

- The FS type in `mkpart` is only a label, it does not create a file system.
- Start at `2048s` for correct alignment.

```bash
mkfs.xfs /dev/vdb1          # XFS (default); ext4: mkfs.ext4
mount /dev/vdb1 /mnt/data   # temporary mount
lsblk --fs                  # devices, file systems, UUIDs
```

Persistent mount in `/etc/fstab` (6 fields):

```
UUID=7a20315d-...   /dbdata   xfs   defaults   0 0
```

- Fields: device (use UUID), mount point (must exist), FS type, options, dump, fsck order (XFS = `0`).
- A wrong entry can break the boot. Check with `findmnt --verify` or `umount` + `mount /mountpoint`.
- After editing: `systemctl daemon-reload`.

### Dev use

```bash
lsblk --fs   # which disk is which, and is it mounted? (before touching any disk)
```

- Always check the device name first: `mklabel` and `mkfs` destroy data. Not from the course.

## 2. Manage Swap Space

- Swap = disk area that extends RAM (holds inactive memory pages). It is much slower than RAM, so it is not a fix for too little RAM.
- Rough size: RAM up to 2 GB = 2x RAM, 2-8 GB = same as RAM, 8-64 GB = at least 4 GB, more than 64 GB = at least 4 GB. For hibernation swap must be bigger than RAM.

Create swap: partition (`linux-swap`), signature, activate.

```bash
parted /dev/vdb mkpart swap1 linux-swap 1001MB 1257MB
udevadm settle
mkswap /dev/vdb2        # prints the UUID
swapon /dev/vdb2        # activate
swapon --show           # check (also: free)
swapoff /dev/vdb2       # deactivate
```

- `swapon -a` activates all swap spaces from `/etc/fstab`.

Persistent swap in `/etc/fstab`:

```
UUID=39e2667a-...   swap   swap   defaults   0 0
```

- Fields: UUID (from `mkswap` or `lsblk --fs`), `swap` instead of a mount point, type `swap`, options, dump `0`, fsck order `0`.
- After editing: `systemctl daemon-reload`.

Priority:

- The kernel uses the highest priority first, then the next. Same priority = round-robin. Default is `-2`.
- Set it with `pri=` in `fstab` (`pri=10` is used before `pri=4`). Check with `swapon --show`.

```
UUID=fbd7fa60-...   swap   swap   pri=10   0 0
```

### Dev use

```bash
free -h   # is the server out of memory and using swap? first thing to check on a slow server
```
