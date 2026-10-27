# RH124 - Chapter 5: Create, View, and Edit Text Files

Short notes for a developer.

Status:
- [x] Redirect output to a file or program
- [ ] Edit text files with vim
- [ ] Change the shell environment
- [ ] Lab

---

## 1. Redirection and pipes

### Three channels

| # | Name | What | Default |
|---|---|---|---|
| 0 | stdin | input | keyboard |
| 1 | stdout | normal output | terminal |
| 2 | stderr | errors | terminal |

### Redirect to a file

| Syntax | Meaning |
|---|---|
| `> file` | stdout to file (overwrites!) |
| `>> file` | stdout to file (appends) |
| `2> file` | only errors to file |
| `2> /dev/null` | throw errors away |
| `> file 2>&1` | stdout + errors to one file |
| `&> file` | same, short form (bash) |
| `&>> file` | same, but append |

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

## 2. vim (to do)

## 3. Shell environment (to do)

---

## Quick cheat sheet

| Task | Command |
|---|---|
| Save output to a file | `cmd > file` |
| Add output to a file | `cmd >> file` |
| Save everything incl. errors | `cmd > file 2>&1` |
| Hide errors | `cmd 2> /dev/null` |
| Count lines | `cmd \| wc -l` |
| Save and show | `cmd \| tee file` |

## Mistakes I want to avoid

- `>` overwrites the file without asking. Use `>>` to add.
- `2>&1 > file` is the wrong order.
- `&>` does not work in a pipe.
- Redirecting in the middle of a pipe stops the data for the next command.
