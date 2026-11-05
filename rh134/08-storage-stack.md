# Chapter 8. Manage Storage Stack

Goal: create and manage logical volumes with file systems or swap, and understand the storage stack layers.

Sections:

1. Create and Extend Logical Volumes
2. Manage Layered Storage

## 1. Create and Extend Logical Volumes

LVM adds a flexible layer on top of physical storage. LVs can be resized without stopping apps or unmounting.

| Level                | Meaning                                                        |
| -------------------- | -------------------------------------------------------------- |
| PV (Physical Volume) | device (partition, disk, RAID) initialized for LVM             |
| VG (Volume Group)    | storage pool from one or more PVs (a PV belongs to one VG)     |
| LV (Logical Volume)  | volume made from free extents of a VG, you format and mount it |

```bash
parted /dev/vdb set 1 lvm on          # partition type "Linux LVM"
pvcreate /dev/vdb1 /dev/vdb2          # PVs
vgcreate vg01 /dev/vdb1 /dev/vdb2     # VG
lvcreate -n lv01 -L 300M vg01         # LV (-L size, -l number of extents)
mkfs -t xfs /dev/vg01/lv01
mount /dev/vg01/lv01 /mnt/data
```

- Persistent mount: `/dev/vg01/lv01 /mnt/data xfs defaults 0 0` in `/etc/fstab`.
- VDO (deduplication + compression): `dnf install vdo kmod-kvdo`, then `lvcreate --type vdo --name vdo-lv01 --size 5G vg01`.
- Show status: `pvs` `vgs` `lvs` (short), `pvdisplay` `vgdisplay` `lvdisplay` (detailed).

Extend:

```bash
pvcreate /dev/vdb3
vgextend vg01 /dev/vdb3                 # add a PV to the VG
lvextend -L +500M /dev/vg01/lv01        # "+" = add to the current size
xfs_growfs /mnt/data                    # XFS (mount point)
resize2fs /dev/vg01/lv01                # ext4 (device)
```

- `lvextend -r` extends the LV and the file system in one step.
- XFS can only grow, never shrink. For swap on an LV: `swapoff`, `lvextend`, `mkswap`, `swapon`.

Reduce a VG and remove LVM storage:

```bash
pvmove -A y /dev/vdb3        # move data to other PVs (back up first!)
vgreduce vg01 /dev/vdb3      # remove the PV from the VG
umount /mnt/data             # remove the fstab entry too
lvremove /dev/vg01/lv01
vgremove vg01
pvremove /dev/vdb1 /dev/vdb2
```

- Removing an LV destroys its data.

### Dev use

```bash
lvextend -r -L +5G /dev/vg01/lv01   # database disk is full: grow the volume and the file system, no downtime
```

- Main practical benefit of LVM. Not from the course.

## 2. Manage Layered Storage

Storage stack (bottom to top): block device, multipath, partitions, RAID, LVM, file system (or raw space).

| Layer        | Meaning                                                                     |
| ------------ | --------------------------------------------------------------------------- |
| Block device | disks, SSD, iSCSI, FCoE, virtio, SAS, NVMe                                  |
| Multipath    | `dm-multipath` joins several paths into one device in `/dev/mapper/`        |
| RAID         | redundancy and speed (LVM supports 0, 1, 4, 5, 6, 10)                       |
| LVM          | PV, VG, LV. Supports LUKS (encryption) and VDO (deduplication, compression) |
| File system  | XFS is recommended. Big databases may use raw space                         |

- LUKS encrypts the whole device, so the file system structure is hidden.

### Stratis

- Local storage manager (Technology Preview in RHEL 9). Pools of disks, file systems from pools.
- Thin provisioning: space is taken from the pool as data is written. Uses Device Mapper and XFS.
- Packages: `stratis-cli`, `stratisd`. Change Stratis file systems only with Stratis commands.

```bash
dnf install stratis-cli stratisd
systemctl enable --now stratisd
stratis pool create pool1 /dev/vdb
stratis pool add-data pool1 /dev/vdc
stratis filesystem create pool1 fs1          # /dev/stratis/pool1/fs1
stratis filesystem snapshot pool1 fs1 snap1
stratis pool list                            # real free space
```

Persistent mount in `/etc/fstab`:

```
UUID=c7b57190-... /dir1 xfs defaults,x-systemd.requires=stratisd.service 0 0
```

- Without `x-systemd.requires=stratisd.service` the boot fails and goes to `emergency.target`.
- Do not use `df` for Stratis (it always shows 1 TiB). Use `stratis pool list`. A full pool silently drops writes.

### Dev use

```bash
stratis pool list   # how much real space is left in the pool? (df lies with thin provisioning)
```

- Same idea as thin provisioning in cloud disks and databases: check the real usage. Not from the course.
