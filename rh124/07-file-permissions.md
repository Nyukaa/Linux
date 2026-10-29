## 1. Reading permissions

```
-rw-rw-r--. 1 student student 0 Mar  8 17:36 test
```

| Part              | Meaning                                    |
| ----------------- | ------------------------------------------ |
| `-`               | file type (`-` file, `d` dir, `l` symlink) |
| `rw-`             | user (owner)                               |
| `rw-`             | group                                      |
| `r--`             | other                                      |
| `student student` | owner, group owner                         |

| Perm | On a file      | On a directory        |
| ---- | -------------- | --------------------- |
| `r`  | read content   | list file names       |
| `w`  | change content | create / delete files |
| `x`  | run as program | `cd` into it          |

- Order: user, then group, then other. The first matching set wins.
- To delete a file you need `w` + `x` on the **directory**, not on the file.
- Permissions are not inherited by subdirectories.
- `ls -ld dir` shows permissions of the directory itself.

## 2. Change permissions and owner

### chmod, symbolic

| Who       |     | Op              |     | Perm                                                 |
| --------- | --- | --------------- | --- | ---------------------------------------------------- |
| `u` user  |     | `+` add         |     | `r` read                                             |
| `g` group |     | `-` remove      |     | `w` write                                            |
| `o` other |     | `=` set exactly |     | `x` execute                                          |
| `a` all   |     |                 |     | `X` execute only for dirs / already-executable files |

```bash
chmod a+x script.sh          # make a script executable
chmod go-rw document.pdf     # remove read+write for group and other
chmod -R g+rwX project/      # recursive; x only where it makes sense
```

### chmod, octal

`r`=4, `w`=2, `x`=1. Digits are: user, group, other.

| Number | Meaning     | Typical use                    |
| ------ | ----------- | ------------------------------ |
| `600`  | `rw-------` | private keys (`~/.ssh/id_rsa`) |
| `644`  | `rw-r--r--` | normal file                    |
| `750`  | `rwxr-x---` | directory for a group          |
| `755`  | `rwxr-xr-x` | normal directory / script      |

### chown, chgrp

```bash
sudo chown user file             # change owner (root only)
sudo chown -R user:group dir     # owner + group, recursive
chown :group file                # change only the group
chgrp group file                 # same, with chgrp
```

- Only root can change the **owner**. The owner can change the **group**, but only to a group they belong to.
- Use `owner:group` with a colon, not a dot.

## 3. Default permissions (umask)

- New file starts at `666`, new directory at `777`.
- A file never gets `x` automatically.
- `umask` removes permission bits from new files and directories.

| umask | New file | New directory |
| ----- | -------- | ------------- |
| `022` | `644`    | `755`         |
| `002` | `664`    | `775`         |
| `027` | `640`    | `750`         |
| `077` | `600`    | `700`         |

```bash
umask          # show current umask
umask 077      # change it for this shell only
```

- Make it permanent: `~/.bashrc` (one user) or a file in `/etc/profile.d/` (all users, needs root).
- If umask is `077`, files I create are private. Good for secrets, bad for shared folders.
