# RH124 - Chapter 13: Access Linux File Systems

---

## 1. File systems and devices

- Default file system: **XFS**. Also: ext4. exFAT for removable media (RHEL 9). GFS2 for shared cluster disks.
- Mount point = an empty directory where a file system is attached. Everything is one tree (no drive letters).
- Block device = file in `/dev` for low-level access to storage. A disk is split into partitions.

| Device                   | Names                            |
| ------------------------ | -------------------------------- |
| SATA / SAS / USB         | `/dev/sda`, `/dev/sda1`          |
| Virtual machine (virtio) | `/dev/vda`, `/dev/vda1`          |
| NVMe SSD                 | `/dev/nvme0n1`, `/dev/nvme0n1p1` |
| SD card                  | `/dev/mmcblk0`, `/dev/mmcblk0p1` |

- LVM: volume group + logical volumes. Path: `/dev/VGNAME/LVNAME`.
- `ls -l /dev/sda1` shows type `b` (block device).

```bash
df -h                  # size, used, free per file system (powers of 1024)
df -H                  # same, in SI units (powers of 1000)
du -h /usr/share       # disk usage per subdirectory
du -sh /etc /boot /var # only the total (-s)
```

- `tmpfs` and `devtmpfs` live in RAM and are empty after reboot.
- Disk full? `df -h` (which file system), then `du -sh` (which folder).

---

df -h # много ли места осталось
du -sh \* # что занимает место в текущей папке
du -sh /var/log # сколько занимают логи

## 2. Mount and unmount

```bash
lsblk                          # disks and partitions
lsblk -fp                      # full paths, file system type, UUID, mount points

sudo mkdir -p /mnt/data        # the mount point must exist
sudo mount /dev/vda4 /mnt/data                                    # by device name
sudo mount UUID="efd314d0-b56e-45db-bbb3-3f32ae98f652" /mnt/data  # by UUID (more stable)

sudo umount /mnt/data          # unmount (command is umount, not unmount)
lsof /mnt/data                 # who uses it? (if "target is busy")
```

- Device names can change. UUID does not (until the file system is re-created).
- Files in a non-empty mount point are hidden while it is mounted.
- A manual mount is gone after reboot. For permanent mounts use `/etc/fstab` (RH134).
- "target is busy": maybe my shell is inside the directory. Run `cd ~`.
- Always unmount a removable drive before unplugging (flushes cached data).
- Removable media in a GUI session: `/run/media/USER/LABEL`.

---

Для разработчика это про подключение дисков, USB и сетевых томов, например дополнительного диска на облачном сервере.
lsblk -f # какие диски и разделы есть
sudo mount /dev/sdb1 /mnt
sudo umount /mnt
lsof +D /mnt # кто мешает (не из курса)

## 3. Locate files

|           | `locate`                          | `find`                                     |
| --------- | --------------------------------- | ------------------------------------------ |
| How       | searches a database               | searches live                              |
| Speed     | instant                           | slower                                     |
| New files | may be missing (until `updatedb`) | always found                               |
| Criteria  | name or path only                 | name, owner, permissions, size, time, type |

```bash
locate passwd              # partial match in the name or path
locate -i messages         # case-insensitive
locate -n 5 passwd         # first 5 results
sudo updatedb              # refresh the database now

find / -name sshd_config           # exact name
find / -name '*.txt'               # wildcard: ALWAYS use quotes
find / -iname '*messages*'         # case-insensitive
find -user developer               # by owner (also -group, -uid, -gid)
find /etc -type d                  # by type: f file, d dir, l symlink, b block
find -size +10M                    # size: k (lowercase), M, G (uppercase); + bigger, - smaller
find / -mmin -60                   # changed less than 60 minutes ago (+ = more than)
find /home -perm -o=w -ls          # others can write; -ls shows details
find / -type f -links +1           # files with more than one hard link
```

| `-perm`      | Meaning                    |
| ------------ | -------------------------- |
| `-perm 764`  | exactly these permissions  |
| `-perm -324` | all of these bits          |
| `-perm /442` | at least one of these bits |

- `find` options use ONE dash (`-name`, not `--name`).
- `-size 1M` rounds up, so it matches files up to 1 MB.
- Hide "Permission denied": `find / -name x 2>/dev/null`.

---

find . -name '\*.log' # логи в текущей папке и глубже
find / -name sshd_config 2>/dev/null # скрыть сообщения Permission denied
find . -type f -size +100M # крупные файлы
find . -type f -mmin -60 # что менялось за последний час
