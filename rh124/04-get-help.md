# RH124 - Chapter 4: Get Help in RHEL

Short guide for a developer. Goal: find the answer in the terminal without Google.

## Quick help: which tool to use?

| I want to... | Use |
|---|---|
| See quick options of a command | `command --help` |
| Read the full docs | `man command` |
| Find a command when I forget the name | `man -k keyword` |
| Get a one-line description | `man -f command` (or `whatis command`) |
| Read about a config file format | `man 5 name` |

```bash
ls --help | less        # quick options, scroll with arrows, q to exit
man ls                  # full manual
man -k password         # which commands/pages are about "password"?
man -f passwd           # passwd (1) and passwd (5)
man 5 passwd            # file format of /etc/passwd, not the command
man 5 crontab           # config format, useful for cron jobs
man ssh_config          # how to write ~/.ssh/config
```

## Inside a man page

| Key | Action |
|---|---|
| `Space` / `b` | page down / page up |
| `/text` | search forward |
| `n` / `Shift+n` | next / previous match |
| `g` / `Shift+g` | top / bottom |
| `q` | quit |

Tip: search for an option, for example `/-r`, to jump straight to it.

## How to read a man page fast

1. **SYNOPSIS**: the syntax. `[ ]` = optional, `...` = can repeat.
2. **EXAMPLES**: if it exists, read it first.
3. **OPTIONS**: use `/` to find the flag you need.
4. **SEE ALSO**: related pages.

## Sections (only the useful ones)

| # | Content |
|---|---|
| 1 | user commands (`man ls`) |
| 5 | file formats and configs (`man 5 passwd`) |
| 8 | admin commands (`man 8 useradd`) |

- `passwd(1)` = the command, `passwd(5)` = the file. The number in brackets is the section.
- `man passwd` opens the first section it finds. Use `man 5 passwd` to choose.

## Search options

- `man -f word`: search page names only.
- `man -k word`: search names and descriptions (best for "what command does X?").
- `man -K word`: search full text of all pages (slow).
- `-k` needs an index made by `mandb` (root). It is updated automatically when packages are installed.

## Notes for me

- On Mac, `man` is the BSD version: flags can differ from RHEL. Check on the RHEL lab machine for the exam.
- Man pages are for the **installed** version of a tool, so they match the system I am on.
