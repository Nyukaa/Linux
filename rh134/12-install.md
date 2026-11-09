# Chapter 12. Install Red Hat Enterprise Linux

Goal: install RHEL on servers and virtual machines.

## 1. Install Red Hat Enterprise Linux

- Installer: Anaconda. Two methods: manual and automated (Kickstart file).

| Media            | Meaning                                                      |
| ---------------- | ------------------------------------------------------------ |
| Binary DVD (ISO) | Anaconda + BaseOS and AppStream repos, no network needed     |
| Boot ISO         | small image, needs a network package source (HTTP, FTP, NFS) |
| QCOW2            | prebuilt disk for a VM (standard KVM format)                 |

- Minimum 10 GiB disk. RHEL 9 architectures: x86-64-v2, ARMv8.0-A, IBM Z (z14), POWER9.
- Image Builder (`composer-cli` or web console) builds custom images.
- RAM (minimum): 1.5 GiB for local media and NFS, 3 GiB for HTTP/HTTPS/FTP (x86-64-v2). POWER9 and ARM need more.

Graphical install: choose language, then INSTALLATION SUMMARY (central screen), then Begin Installation, then Reboot.

| Item                     | Purpose                                              |
| ------------------------ | ---------------------------------------------------- |
| Time & Date              | time zone (set it even with NTP)                     |
| Connect to Red Hat       | register, system purpose (needs network first)       |
| Software Selection       | base environment (Minimal Install = essentials only) |
| Installation Destination | disks and partitioning (default: automatic)          |
| KDUMP                    | kernel crash dump                                    |
| Security Profile         | restrictions of the chosen profile                   |
| Root Password            | **required**                                         |
| User Creation            | optional, recommended                                |

Troubleshooting during install:

| Keys                   | Content                                             |
| ---------------------- | --------------------------------------------------- |
| `Ctrl+Alt+F1`          | tmux console (5 windows)                            |
| `Ctrl+B 2`             | root shell                                          |
| `Ctrl+B 3` / `4` / `5` | `/tmp/anaconda.log` / `storage.log` / `program.log` |
| `Ctrl+Alt+F6`          | Anaconda graphical interface                        |

- In tmux: press `Ctrl+B`, release, then the number.

### Dev use

```
Software Selection: Minimal Install   # small server: fewer packages, smaller attack surface
```

- Add only what the app needs. Not from the course.

## 2. Automate Installation with Kickstart

- Kickstart = text answer file for Anaconda (unattended install). Commands first, then sections that start with `%` and end with `%end`.

| Section     | Purpose                                                           |
| ----------- | ----------------------------------------------------------------- |
| `%packages` | packages: `name`, `@group`, `@^environment`, `-package` (exclude) |
| `%pre`      | script before partitioning (avoid it, hard to debug)              |
| `%post`     | script after installation (`--interpreter=` for Python)           |

- Required commands: `lang`, `keyboard`, `timezone`, `rootpw`, `bootloader`.
- Missing required values: the installer asks or aborts.
- Alternative: Image Builder (prebuilt images, also for clouds).

```
text
url --url="http://classroom.example.com/content/rhel9.0/x86_64/dvd/"
clearpart --all --initlabel
part / --fstype="xfs" --size=10000
lang en_US
keyboard --vckeymap=us
rootpw --plaintext redhat
timezone America/New_York --utc
selinux --enforcing
reboot

%packages
@core
chrony
-plymouth
%end

%post
echo "Deployed with Kickstart on $(date)" > /etc/motd
%end
```

Steps:

1. Create the file: Kickstart Generator website, or edit `/root/anaconda-ks.cfg` (created by every install).
2. Publish it: FTP/HTTP/NFS server (best), USB/CD-ROM, or local disk.
3. Boot Anaconda with `inst.ks=LOCATION` (press `Tab` on physical machines).

```
inst.ks=http://server/dir/file
inst.ks=nfs:server:/dir/file
inst.ks=hd:device:/dir/file
inst.ks=cdrom:device
```

```bash
ksvalidator /tmp/anaconda-ks.cfg    # syntax check (not URLs, packages, scripts)
ksverdiff -f RHEL8 -t RHEL9         # syntax changes between versions
```

- Both tools are in the `pykickstart` package.

### Dev use

```
%post
echo "Deployed with Kickstart on $(date)" > /etc/motd
%end
```

- Same idea as infrastructure as code: one file builds identical servers. Not from the course.

## 3. Install and Configure Virtual Machines

- KVM (Kernel-based Virtual Machine) = full virtualization built into the Linux kernel. Runs Linux and Windows guests.
- Manage with `virsh` (`libvirt-client`) or the web console Virtual Machines plug-in (`cockpit-machines`). Create from CLI: `virt-install`.
- Also in: Red Hat Virtualization (RHV), OpenStack Platform (RHOSP), OpenShift Virtualization.
- sVirt (SELinux) isolates guests: each VM process gets a unique label, its disks get the same label.
- Machine types: i440FX (old), Q35 (use it, supports PCIe and SATA). UEFI and Secure Boot for VMs: OVMF (`edk2-ovmf`).

```bash
dnf group install "Virtualization Host"
virt-host-validate        # all checks must PASS (needs Intel VT-x or AMD-V)
```

```bash
virt-install --name demo --memory 4096 --vcpus 2 --disk size=40 \
--os-type linux --cdrom /root/rhel.iso
```

Web console:

```bash
dnf install cockpit-machines
systemctl enable --now cockpit.socket     # then open https://localhost:9090
```

- Virtual Machines, Create VM, fill the form, Create, then Install.
- Name = VM configuration name (not the hostname). Reboot once after installing the plug-in so `libvirt` starts.

### Dev use

```bash
virt-install --name demo --memory 4096 --vcpus 2 --disk size=40 --os-type linux --cdrom /root/rhel.iso
```

- A throwaway VM for testing deploy scripts or Kickstart files. Not from the course.
