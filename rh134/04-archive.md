# Chapter 4. Archive and Transfer Files

Goal: archive and copy files from one system to another.

Sections:

1. Manage Compressed tar Archives
2. Transfer Files Between Systems Securely
3. Synchronize Files Between Systems Securely

## 1. Manage Compressed tar Archives

- An archive = one file that contains many files. `tar` creates, lists and extracts archives, with optional compression.
- One action is required: `-c` create, `-t` list, `-x` extract. `-f` = archive file name.
- Useful: `-v` verbose, `-p` keep permissions (default for root), `--acls` `--selinux` `--xattrs` (not saved by default).

| Option | Algorithm | Suffix     | Note                                   |
| ------ | --------- | ---------- | -------------------------------------- |
| `-z`   | gzip      | `.tar.gz`  | fastest, widely available              |
| `-j`   | bzip2     | `.tar.bz2` | smaller than gzip                      |
| `-J`   | xz        | `.tar.xz`  | best compression                       |
| `-a`   | by suffix |            | picks the algorithm from the file name |

```bash
tar -czf /root/etcbackup.tar.gz /etc   # create (gzip)
tar -tf /root/etcbackup.tar.gz         # list (no compression option needed)
mkdir /root/etcbackup && cd /root/etcbackup
tar -xf /root/etcbackup.tar.gz         # extract into an empty directory
```

- `tar` removes the leading `/` from absolute paths, so extraction does not overwrite existing files.
- You need read permission on the files you archive.
- Wrong compression option on extract gives an error (`not in gzip format`).
- `gzip`, `bzip2`, `xz` compress a single file (`gunzip`, `bunzip2`, `unxz` to uncompress). `gzip -l` and `xz -l` show the uncompressed size.

### Dev use

```bash
tar -czf project.tar.gz --exclude=node_modules myproject/   # archive a project without node_modules
```

- `--exclude` is not from the course.

## 2. Transfer Files Between Systems Securely

- `sftp` and `scp` are part of OpenSSH: authentication and encrypted transfer.
- Remote address format: `[user@]host:/path` (no user = your local username).

```bash
sftp remoteuser@remotehost    # interactive session (sftp> prompt)
```

| In sftp                    | Meaning                         |
| -------------------------- | ------------------------------- |
| `ls`, `cd`, `mkdir`, `pwd` | run on the remote host          |
| `lpwd`, `lcd`, `lls`       | run on the local host (add `l`) |
| `put file` / `put -r dir`  | upload a file / a directory     |
| `get file`                 | download a file                 |
| `exit`                     | quit                            |

- One-line download (no interactive session): `sftp remoteuser@remotehost:/home/remoteuser/remotefile`. You cannot upload this way.
- Since RHEL 9, `scp` uses the `sftp` protocol. The old protocol had a code injection issue. `scp -O` returns to it: do not use it.

### Dev use

```bash
scp ./build.tar.gz user@server:/tmp/   # copy a file to a server
```

- Quick way to move a build to a server. Not from the course.

## 3. Synchronize Files Between Systems Securely

- `rsync` copies files securely and efficiently: after the first sync it copies only the differences.
- Remote address: `[user@]host:/path`. One side must be local. Two local directories also work.
- To keep file owners you must be root on the destination.

| Option      | Meaning                                                                     |
| ----------- | --------------------------------------------------------------------------- |
| `-n`        | dry run (shows what would happen)                                           |
| `-v`        | verbose                                                                     |
| `-a`        | archive: `-rlptgoD` (recursive, links, perms, times, group, owner, devices) |
| `-H`        | keep hard links (not in `-a`)                                               |
| `-A` / `-X` | keep ACLs / SELinux contexts (not in `-a`)                                  |

```bash
rsync -av /var/log hosta:/tmp      # local -> remote
rsync -av hosta:/var/log /tmp      # remote -> local
rsync -avn /var/log hosta:/tmp     # dry run first
```

Trailing slash on the source:

- `/var/log` -> creates `/tmp/log` (the directory itself).
- `/var/log/` -> copies only the contents into `/tmp`.

### Dev use

```bash
rsync -avn ./dist/ user@server:/var/www/app/   # dry run, then remove -n to deploy
```

- Deploy a static build to a server. Not from the course.
