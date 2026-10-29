## 1. Process states and lifecycle

- A process is a running program. It has a PID and a parent (PPID).
- `fork` creates a child. `systemd` (PID 1) is the ancestor of all processes.
- Zombie: the child has finished, but the parent has not cleaned it up yet.

| Flag | State                                   |
| ---- | --------------------------------------- |
| `R`  | running or ready to run                 |
| `S`  | sleeping (waits for an event)           |
| `D`  | uninterruptible sleep (ignores signals) |
| `T`  | stopped                                 |
| `Z`  | zombie                                  |

```bash
ps                 # my processes in this terminal
ps aux             # all processes (BSD style, no dash)
ps -ef             # all processes (UNIX style, has PPID)
ps --forest        # tree of parents and children
```

- `ps aux` is not the same as `ps -aux`.
- State is in the `STAT` column of `ps` and the `S` column of `top`.

## 2. Kill processes

| Signal          | Meaning                       |
| --------------- | ----------------------------- |
| `TERM` (15)     | polite stop, default          |
| `INT` (2)       | `Ctrl+c`                      |
| `TSTP` (20)     | `Ctrl+z` (suspend)            |
| `KILL` (9)      | force kill, cannot be caught  |
| `HUP` (1)       | reload config (many services) |
| `STOP` / `CONT` | pause / resume                |

```bash
pgrep -l -u bob          # find processes (does not kill)
kill PID                 # send TERM
kill -9 PID              # send KILL (last resort)
kill -l                  # list signals
kill %1                  # kill background job 1 (see jobs)
killall name             # all processes with this command name
pkill -u bob             # all processes of a user
pkill -t tty3            # all processes on a terminal
pkill -P PID             # children of a parent
pstree -p                # process tree
w                        # who is logged in
```

- Order: `TERM` first, then `INT`, then `KILL` only if needed (no cleanup with `KILL`).
- A stopped (`T`) process ignores `TERM`. Use `kill -CONT PID` or `kill -9 PID`.
- To kill a user's login shell too, use `SIGKILL`.

## 3. Monitor process activity

- Load average counts processes that are running (`R`) or waiting for I/O (`D`).
- `uptime` shows 3 numbers: last 1, 5 and 15 minutes.
- Divide by the number of CPUs (`lscpu`). Below 1 = OK. Above 1 = overloaded.
- High load but low CPU? Check disk and network.

```bash
uptime        # load average
lscpu         # number of CPUs
top           # live view
```

| Key in `top` | Action         |
| ------------ | -------------- |
| `Shift+p`    | sort by CPU    |
| `Shift+m`    | sort by memory |
| `u`          | filter by user |
| `k`          | kill a process |
| `r`          | renice         |
| `1`          | show each CPU  |
| `q`          | quit           |

- `VIRT` in `top` = `VSZ` in `ps`. `RES` in `top` = `RSS` in `ps`.
