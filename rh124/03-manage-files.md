# Chapter 3: Manage Files from the Command Line

My own notes. Goal: copy, move, create, delete and organize files from the Bash shell.

## 1. File-system hierarchy

- Linux has one tree. The top is `/` (root directory).
- `/` is also the separator in paths: `/etc/issue`.

| Directory  | What is inside                                                   |
| ---------- | ---------------------------------------------------------------- |
| `/boot`    | files to start the system                                        |
| `/dev`     | device files (access to hardware)                                |
| `/etc`     | system configuration                                             |
| `/home`    | home directories of normal users                                 |
| `/root`    | home directory of the root user                                  |
| `/run`     | runtime data (PID and lock files), cleared on reboot             |
| `/tmp`     | temporary files, old files deleted after 10 days                 |
| `/var/tmp` | temporary files, old files deleted after 30 days                 |
| `/usr`     | installed software, libraries, read-only data                    |
| `/var`     | data that changes but stays after reboot: logs, databases, cache |

Inside `/usr`: `bin` (user commands), `sbin` (admin commands), `local` (local software).

Since RHEL 7: `/bin`, `/sbin`, `/lib`, `/lib64` are symbolic links to the same folders in `/usr`.

Content types: static, dynamic (variable), persistent, runtime.

---

## 2. Paths

- **Absolute path**: starts with `/`. Example: `/var/log/messages`.
- **Relative path**: does not start with `/`. It starts from the current directory. Example: `log/messages` from `/var`.
- Linux file names are **case-sensitive**: `File.txt` and `file.txt` are different files.
- Spaces in names: use quotes (`"my file.txt"`), or better, avoid spaces.

### Navigation

```bash
pwd          # where am I?
ls           # what is here?
cd Videos    # go to a directory
cd           # go home
cd -         # go to the previous directory
cd ..        # go up one level
cd .         # stay here
ls -l ~      # ~ always means my home directory
```

### Useful `ls` options

```bash
ls -l    # long format (permissions, owner, size, date)
ls -a    # show hidden files too
ls -R    # recursive
ls -la   # combine options
```

- `.` = current directory, `..` = parent directory.
- Files that start with `.` are hidden. This is not security. Real protection = file permissions.

### `touch`

```bash
touch file.txt   # create an empty file (or update its timestamp)
```

---

## 3. Create, copy, move, remove

### mkdir

```bash
mkdir ProjectX ProjectY ProjectZ              # several directories
mkdir -p Thesis/Chapter1 Thesis/Chapter2      # -p creates missing parents
```

- Without `-p`, `mkdir` fails if the parent does not exist. This is good: it catches typos.
- Be careful with `-p`: a typo like `Video/Watched` creates a wrong `Video` folder with no error.

### cp

```bash
cp file1 file2                  # copy a file
cp file1 file2 target_dir/      # copy many files; last argument must be a directory
cp -r Thesis/ ProjectY/         # copy a directory with its content (-r is required)
cp -r ../Thesis/ .              # copy into the current directory
```

- Without `-r`, `cp` skips directories and prints a message.
- If a file with the same name exists in the target, it is overwritten.

### mv

```bash
mv old_name new_name            # rename
mv file.txt Thesis/Chapter1/    # move
mv -v file.txt Thesis/          # -v shows what happened
```

- Move and rename are the same operation: the path of the file changes.

### rm and rmdir

```bash
rm file.txt        # remove a file
rm -r Thesis/      # remove a directory and everything inside
rm -ri Thesis/     # ask before every delete
rm -rf Thesis/     # force, no questions (dangerous)
rmdir ProjectZ     # remove an EMPTY directory only
```

- **No trash bin and no undo** in the shell. Deleted = gone.
- Before using `rm` with a relative path, check `pwd`.
- Tip: run `ls pattern*` first to see what `rm pattern*` will delete.
- If you use `-i` and `-f` together, `-f` wins.

---

## 4. Links (short)

- **Hard link**: `ln file link`. Another name for the same data (same inode, check with `ls -li`). Only files, only the same file system. Data stays until the last link is deleted.
- **Symlink**: `ln -s target link`. A pointer to a name. Works for directories and across file systems. If the target is deleted, the link is dangling. In `ls -l` it shows `l` and `->`.
- `cd -P link` goes to the real directory.

---

## 5. Shell expansions (short)

```bash
ls *.txt                    # * = any string
ls song?.mp3                # ? = one character
ls [ac]*                    # starts with a or c
touch file{1..3}.txt        # brace expansion: file1.txt file2.txt file3.txt
mkdir -p project/{src,docs} # several directories at once
cp config.yml{,.bak}        # quick backup
NAME=anna                   # no spaces around =
echo $NAME                  # variable
echo Today is $(date +%A)   # command substitution (not backticks)
echo "Home: $HOME"          # double quotes: variables still work
echo 'Home: $HOME'          # single quotes: literal text
echo \$HOME                 # \ protects one character
```

- If a pattern matches nothing, it is passed to the command as plain text.

---

## Quick cheat sheet

| Task                      | Command                      |
| ------------------------- | ---------------------------- |
| Current directory         | `pwd`                        |
| List files (all, long)    | `ls -la`                     |
| Go home / back / up       | `cd` / `cd -` / `cd ..`      |
| Create empty file         | `touch file`                 |
| Create nested directories | `mkdir -p a/b/c`             |
| Copy file / directory     | `cp a b` / `cp -r dir1 dir2` |
| Move or rename            | `mv a b`                     |
| Delete file / directory   | `rm a` / `rm -r dir`         |
| Delete empty directory    | `rmdir dir`                  |

## Mistakes I want to avoid

- `cp` on a directory without `-r` does nothing.
- `rm -r` has no undo.
- `mkdir -p` with a typo creates a wrong folder silently.
- Paths with spaces need quotes.
