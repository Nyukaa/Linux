# RH124 - Chapter 5: Create, View, and Edit Text Files

---

## 1. Redirection and pipes

### Three channels

| #   | Name   | What          | Default  |
| --- | ------ | ------------- | -------- |
| 0   | stdin  | input         | keyboard |
| 1   | stdout | normal output | terminal |
| 2   | stderr | errors        | terminal |

### Redirect to a file

| Syntax         | Meaning                      |
| -------------- | ---------------------------- |
| `> file`       | stdout to file (overwrites!) |
| `>> file`      | stdout to file (appends)     |
| `2> file`      | only errors to file          |
| `2> /dev/null` | throw errors away            |
| `> file 2>&1`  | stdout + errors to one file  |
| `&> file`      | same, short form (bash)      |
| `&>> file`     | same, but append             |

- `/dev/null` is a black hole: everything sent there disappears.
- `> file` does not hide errors. They still go to the terminal.
- Order matters: `> file 2>&1` is correct. `2>&1 > file` leaves errors on the screen.

```bash
date > /tmp/saved-timestamp
echo "new line" >> notes.txt
find /etc -name passwd 2> /dev/null
find /etc -name passwd > out.txt 2> errors.txt
find /etc -name passwd &> all.txt
```

### Pipes

`|` sends stdout of one command to stdin of the next one.

```bash
ls | wc -l                      # how many files?
ls -l /usr/bin | less           # read long output page by page
ls -t | head -n 10              # 10 newest files
```

- Redirection works with **files**. Pipe works with **commands**.
- Trap: `ls > file | less` shows nothing, because the output went to the file.

### tee (save and pass on)

```bash
ls -l | tee out.txt | less        # save and keep going
npm run build 2>&1 | tee build.log   # see output and save a log
ls -l | tee -a out.txt            # -a = append
```

- In a pipe, use `2>&1`, not `&>`:
  ```bash
  find / -name passwd 2>&1 | less
  ```

---

## 2. vim

Why: vim is on almost every server. I need it to edit configs over ssh.

| Mode              | How to enter               |
| ----------------- | -------------------------- |
| Command (default) | `Esc`                      |
| Insert            | `i`                        |
| Visual            | `v` / `Shift+v` / `Ctrl+v` |
| Extended command  | `:`                        |

| Key          | Action                                       |
| ------------ | -------------------------------------------- |
| `i` / `Esc`  | start typing / back to command mode          |
| `u`          | undo                                         |
| `x`          | delete a character                           |
| `h j k l`    | left, down, up, right                        |
| `a` / `A`    | append after cursor / at end of line         |
| `y` then `p` | copy and paste (in visual mode select first) |
| `:w`         | save                                         |
| `:wq`        | save and quit                                |
| `:q!`        | quit without saving                          |

- Lost? Press `Esc` a few times.
- `vimtutor` is a built-in tutorial.
- `~/.vimrc` example for YAML:

```
  autocmd FileType yaml setlocal ts=2
  set number
```

## 3. Shell environment

```bash
NAME=value            # shell variable (this shell only, no spaces around =)
echo $NAME            # use it, ${NAME} if text follows
export EDITOR=vim     # environment variable (programs can see it)
export PATH=${PATH}:/home/user/bin   # add a folder to PATH
env                   # list environment variables
unset NAME            # remove a variable
alias ll='ls -la'     # shortcut (put in ~/.bashrc)
unalias ll
```

| File              | When it runs                           |
| ----------------- | -------------------------------------- |
| `~/.bash_profile` | login shell (ssh)                      |
| `~/.bashrc`       | every interactive shell (new terminal) |

- Put variables and aliases in `~/.bashrc`, then run `source ~/.bashrc`.
- `PATH` = folders where the shell looks for commands (separated by `:`).
- `PS1` changes the prompt. `HISTTIMEFORMAT="%F %T "` adds time to `history`.

---

## Quick cheat sheet

| Task                         | Command            |
| ---------------------------- | ------------------ |
| Save output to a file        | `cmd > file`       |
| Add output to a file         | `cmd >> file`      |
| Save everything incl. errors | `cmd > file 2>&1`  |
| Hide errors                  | `cmd 2> /dev/null` |
| Count lines                  | `cmd \| wc -l`     |
| Save and show                | `cmd \| tee file`  |

## Mistakes I want to avoid

- `>` overwrites the file without asking. Use `>>` to add.
- `2>&1 > file` is the wrong order.
- `&>` does not work in a pipe.
- Redirecting in the middle of a pipe stops the data for the next command.
