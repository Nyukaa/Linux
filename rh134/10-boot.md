# Chapter 10. Control the Boot Process

Goal: manage the boot process to control services and to troubleshoot and repair problems.

Sections:

1. Select the Boot Target
2. Reset the Root Password
3. Repair File-system Issues at Boot

## 1. Select the Boot Target

Boot process: firmware (UEFI/BIOS, POST) -> GRUB2 -> kernel + initramfs -> `systemd` (PID 1) -> default target.

- GRUB2 config: `/boot/grub2/grub.cfg` (BIOS), `/boot/efi/EFI/redhat/grub.cfg` (UEFI). Settings: `/etc/default/grub`, `/etc/grub.d/`. Generate with `grub2-mkconfig`.
- Do not run `grub2-install` on UEFI (it creates `grubx64.efi` without Secure Boot signatures).
- initramfs: archive with kernel modules and a small root file system with `systemd`. Tools: `dracut`, `lsinitrd`.
- The real root is mounted at `/sysroot`, then `systemd` starts the default target.

```bash
systemctl poweroff    # stop services, unmount, power off
systemctl reboot
```

| Target              | Purpose                               |
| ------------------- | ------------------------------------- |
| `graphical.target`  | multi-user, graphical and text logins |
| `multi-user.target` | multi-user, text login only           |
| `rescue.target`     | single-user system for repairs        |
| `emergency.target`  | most minimal system (if rescue fails) |

```bash
systemctl list-units --type=target --all
systemctl isolate multi-user.target     # switch now (needs AllowIsolate=yes)
systemctl get-default
systemctl set-default graphical.target  # changes the default.target symlink
```

Different target for one boot only:

1. Reboot, press any key (not Enter) in the GRUB menu.
2. Select the kernel entry, press `e`.
3. Go to the line that starts with `linux`.
4. Append `systemd.unit=rescue.target` (or `emergency.target`).
5. Press `Ctrl+x`.

### Dev use

```bash
systemctl set-default multi-user.target   # server without GUI: saves RAM and CPU
```

## 2. Reset the Root Password

Reset a lost root password from the boot loader (no external media):

1. Reboot, press any key (not Enter) in the GRUB menu.
2. Select the **rescue kernel** entry (the normal kernel asks for the root password).
3. Press `e`, go to the line that starts with `linux`, append `rd.break`.
4. Press `Ctrl+x`, press Enter for maintenance.

Root file system is mounted read-only on `/sysroot`:

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel     # SELinux is not active yet, /etc/shadow has no context
exit                    # leave chroot
exit                    # leave the debug shell, boot continues (full relabel, then reboot)
```

- Cloud images have no rescue kernel: use the default kernel with `rd.break` (no root password needed). The shell appears on the last `console=` argument.

Troubleshoot boot:

| Tool                                              | Use                                                        |
| ------------------------------------------------- | ---------------------------------------------------------- |
| `journalctl -b -1 -p err`                         | errors from the previous boot (needs a persistent journal) |
| `systemctl enable debug-shell.service`            | root shell on TTY9 (`Ctrl+Alt+F9`) early in boot           |
| `systemd.debug-shell` (kernel line)               | same, for one boot                                         |
| `systemd.unit=rescue.target` / `emergency.target` | rescue or emergency shell (needs the root password)        |
| `systemctl list-jobs`                             | find stuck jobs (`running` blocks `waiting`)               |

- Debug shell is an unauthenticated root shell: disable `debug-shell.service` after debugging.
- `emergency.target` mounts root read-only: `mount -o remount,rw /` before editing `/etc/fstab`.

### Dev use

```bash
journalctl -b -1 -p err   # server did not come back after a reboot: what failed in the previous boot?
```

## 3. Repair File-system Issues at Boot

- At boot `systemd` mounts file systems from `/etc/fstab`. A wrong entry or a corrupted file system can stop the boot and open an emergency shell (needs the root password).
- Corrupted file system: `systemd` tries to repair it, else emergency shell.
- Nonexistent device or UUID: timeout, then emergency shell.
- Missing mount point: RHEL 9 creates it automatically.

Repair steps in the emergency shell:

```bash
mount                        # is / mounted "ro"?
mount -o remount,rw /        # make it read/write to edit /etc/fstab
mount --all                  # mount everything from fstab, shows errors
# fix the cause: typo, wrong device or UUID, missing directory (mkdir)
systemctl daemon-reload      # systemd rebuilds .mount units from fstab
mount --all
systemctl reboot             # final test
```

- `nofail` option in an `fstab` entry: the system boots even if this mount fails. Good for quick tests, not for production file systems (an app could start without its data).

### Dev use

```
UUID=...  /data  xfs  defaults,nofail  0 0   # optional disk: a missing disk must not block the boot
```

- Use it only for non-critical disks. Not from the course.
