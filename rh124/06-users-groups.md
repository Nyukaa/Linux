# RH124 - Chapter 6: Manage Local Users and Groups

---

## 1. Users and groups: concepts

### Users

- Every process runs as a user. Every file has an owner. This controls who can access what.
- The system uses the number **UID**. The name is for humans.

| Type          | Details                                      |
| ------------- | -------------------------------------------- |
| Superuser     | `root`, UID 0, full access                   |
| System users  | for services (daemons), no interactive login |
| Regular users | daily work, limited rights                   |

```bash
id               # who am I: uid, gid, all groups
id user02        # info about another user
ls -l file       # 3rd column = owner of the file
ls -ld dir       # owner of the directory itself
ps -au           # processes and their users
```

### /etc/passwd (7 fields, separated by `:`)

```
user01:x:1000:1000:User One:/home/user01:/bin/bash
```

| #   | Field          | Meaning                                |
| --- | -------------- | -------------------------------------- |
| 1   | `user01`       | username                               |
| 2   | `x`            | password placeholder (not stored here) |
| 3   | `1000`         | UID                                    |
| 4   | `1000`         | GID of the **primary group**           |
| 5   | `User One`     | comment / real name                    |
| 6   | `/home/user01` | home directory                         |
| 7   | `/bin/bash`    | login shell                            |

- Shell `/sbin/nologin` = no interactive login (used by system users).

### Groups

- A group is a set of users who share access to files. Identified by **GID**.
- `/etc/group` has 4 fields:

```
group01:x:10000:user01,user02,user03
```

| #   | Field        | Meaning                 |
| --- | ------------ | ----------------------- |
| 1   | `group01`    | group name              |
| 2   | `x`          | password placeholder    |
| 3   | `10000`      | GID                     |
| 4   | `user01,...` | members (supplementary) |

### Primary and supplementary groups

- Every user has **exactly one primary group** (GID in `/etc/passwd`). It owns the files the user creates.
- A new regular user gets a group with the same name (**User Private Group**).
- **Supplementary groups** are extra groups (stored in `/etc/group`). A user can have many.
- Access is allowed if **any** of the user's groups has the right.

```
uid=1001(user01) gid=1003(user01) groups=1003(user01),10(wheel),10000(group01)
```

---

## 2. Superuser access

- Log in as a normal user. Use root rights only when needed.

|             | `su`          | `su -`        | `sudo`        |
| ----------- | ------------- | ------------- | ------------- |
| Environment | mine          | new user's    | mine          |
| Password    | target user's | target user's | **my own**    |
| Logged      | only `su`     | only `su`     | every command |

```bash
su - user02              # login shell as user02
su -                     # become root (needs root password)
sudo command             # run one command as root
sudo -i                  # root shell with login scripts
sudo -s                  # root shell without login scripts
sudo usermod -L user02   # example
sudo tail /var/log/secure   # sudo log
```

- Members of the `wheel` group can use sudo for any command (RHEL 7+).
- Edit sudo rules only with `visudo`. Put extra rules in `/etc/sudoers.d/`:

```
  %wheel   ALL=(ALL:ALL)   ALL            # % = group
  user01   ALL=(ALL)       ALL
  ansible  ALL=(ALL)       NOPASSWD: ALL  # no password, only with SSH keys
```

## 3. User accounts

```bash
sudo useradd user01                 # create user (+ home + private group)
sudo passwd user01                  # set password (no password = no login)
sudo usermod -aG wheel user01       # ADD to a group (keep other groups)
sudo usermod -s /bin/zsh user01     # change shell
sudo usermod -L user01              # lock account
sudo usermod -U user01              # unlock account
sudo userdel user01                 # delete user, keep home
sudo userdel -r user01              # delete user and home
find / -nouser -o -nogroup          # find files with no owner
```

- `usermod -G` WITHOUT `-a` replaces all supplementary groups. Use `-aG`.
- Locking is safer than deleting: deleted UIDs can be reused and the new user gets old files.
- `/etc/login.defs` = defaults for NEW users only.

| UID   | Used for        |
| ----- | --------------- |
| 0     | root            |
| 1-999 | system accounts |
| 1000+ | regular users   |

## 4. Group accounts

```bash
sudo groupadd consultants            # create a group
sudo groupadd -g 35000 consultants   # create with a chosen GID
sudo groupmod -n newname consultants # rename a group
sudo groupmod -g 35001 consultants   # change GID
sudo groupdel consultants            # delete a group
sudo usermod -aG consultants user01  # add user to a group (keep other groups)
sudo useradd -G consultants user02   # new user with a supplementary group
id user01                            # check groups of a user
```

- Always `-aG` when adding a user to a group. `-G` alone replaces all supplementary groups.
- You cannot delete a group that is the primary group of a user.
- Give a whole group sudo rights with one file in `/etc/sudoers.d/`:

```
  %consultants  ALL=(ALL) ALL
```

## 5. Passwords

- Password hashes are in `/etc/shadow` (only root can read it). 9 fields.
- Hash format: `$6$salt$hash`. `6` = SHA-512, `5` = SHA-256, `1` = MD5.
- The salt protects against pre-computed hash lists.

```bash
sudo chage -l user01              # show password policy
sudo chage -M 90 user01           # max days before password must change
sudo chage -m 0 user01            # min days between changes
sudo chage -W 7 user01            # warning days
sudo chage -I 14 user01           # inactive days before lock
sudo chage -E 2026-12-31 user01   # account expires on this date
sudo chage -d 0 user01            # must change password at next login
sudo usermod -L -e 2026-12-31 user01   # lock and expire
sudo usermod -s /sbin/nologin svc      # no interactive login
```

- Defaults for NEW users only: `/etc/login.defs` (`PASS_MAX_DAYS`, `PASS_MIN_DAYS`, `PASS_WARN_AGE`) and `/etc/default/useradd` (`INACTIVE`, `EXPIRE`).
- `nologin` blocks shell login, but the user can still use apps (web, FTP, mail) with the password.

## Quick cheat sheet

| Task                           | Command                               |
| ------------------------------ | ------------------------------------- |
| My user, UID, groups           | `id`                                  |
| Info about another user        | `id username`                         |
| Owner of a file                | `ls -l file`                          |
| Processes with users           | `ps -au`                              |
| Root shell                     | `sudo -i`                             |
| Create user + set password     | `useradd user` then `passwd user`     |
| Add user to a group            | `usermod -aG group user`              |
| Lock / unlock user             | `usermod -L user` / `usermod -U user` |
| Delete user and home           | `userdel -r user`                     |
| Create group                   | `groupadd group`                      |
| Show password policy           | `chage -l user`                       |
| Force password change at login | `chage -d 0 user`                     |
| Set account expiry             | `chage -E YYYY-MM-DD user`            |
| Edit sudo rules                | `visudo` or file in `/etc/sudoers.d/` |

## Dev notes

- "Permission denied" is often about owner or group. Check with `id` and `ls -l`.
- Do not work as root if a normal user is enough. Use `sudo`.
- Lock accounts instead of deleting them.
- `usermod -G` without `-a` removes the user from all other groups.
