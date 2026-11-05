# Chapter 9. Access Network-Attached Storage

Goal: access network-attached storage with NFS.

Sections:

1. Manage Network-Attached Storage with NFS
2. Automount Network-Attached Storage

## 1. Manage Network-Attached Storage with NFS

- NFS server exports directories, client mounts them. RHEL 9 default: NFS 4.2. NFSv4 uses only TCP, NFSv3 uses TCP or UDP.
- Client tools: `dnf install nfs-utils`. Mounting needs a privileged user.
- Ways to mount: manually (`mount`), at boot (`/etc/fstab`), on demand (automounter).

See what a server exports:

| Version | How                                                                                          |
| ------- | -------------------------------------------------------------------------------------------- |
| NFSv3   | `showmount --exports server` (needs `rpcbind`, port 111)                                     |
| NFSv4   | `mount server:/ /mountpoint`, then `ls /mountpoint` (export tree, nothing is really mounted) |

- `showmount` times out on an NFSv4-only server.

```bash
mkdir /mountpoint
mount -t nfs -o rw,sync server:/export /mountpoint   # temporary
```

- `rw` = read/write. `sync` = synchronous writes (recommended for production).
- Do not use `/mnt` for permanent mounts.

Persistent mount in `/etc/fstab`:

```
server:/export  /mountpoint  nfs  rw  0 0
```

```bash
mount /mountpoint     # options are taken from fstab
umount /mountpoint    # does not remove the fstab entry
```

- "device is busy": `cd` out of the mount point, find processes with `lsof /mountpoint`, close them, retry. Last resort: `umount -f` (can lose unwritten data).

### Dev use

```bash
mount -t nfs -o rw,sync server:/export /mountpoint   # shared uploads or media between app servers
```

- Typical use: several servers use the same folder. Not from the course.

## 2. Automount Network-Attached Storage

- Automounter (`autofs`) mounts file systems on demand and unmounts them when they are not used. Idle file systems use almost no resources. Normal users do not need `mount`.
- Packages: `dnf install autofs nfs-utils`. Start: `systemctl enable --now autofs`.
- Direct map: fixed absolute mount point. Indirect map: mount point under a base directory, created on demand.

Two levels of files:

1. Master map: file in `/etc/auto.master.d/` with the `.autofs` extension. Format: `mount-point  map-file`.
2. Map file (usually `/etc/auto.name`). Format: `mount-point  mount-options  source-location`.

```
# /etc/auto.master.d/indirect.autofs
/shares  /etc/auto.indirect

# /etc/auto.indirect
work  -rw,sync  hosta:/shares/work     # mounts at /shares/work
```

```
# /etc/auto.master.d/direct.autofs
/-  /etc/auto.direct

# /etc/auto.direct
/mnt/docs  -rw,sync  hosta:/shares/docs
```

Wildcard (one entry for all subdirectories):

```
# /etc/auto.indirect
*  -rw,sync  hosta:/shares/&
```

- Options start with `-`, comma-separated, no spaces. Useful: `-fstype=nfs4`, `-strict`.
- If the server exports read-only, the client gets read-only.
- Alternative: `x-systemd.automount` option in `/etc/fstab`, then `systemctl daemon-reload` and `systemctl start remote-finance.automount` (for `/remote/finance`). Only absolute paths.

### Dev use

```
# /etc/auto.master.d/direct.autofs
/-  /etc/auto.direct
```

- Share mounts only when the app really uses them, so a dead NFS server does not block the boot. Not from the course.
