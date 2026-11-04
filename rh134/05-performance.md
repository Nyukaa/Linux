# Chapter 5. Tune System Performance

Goal: improve performance with tuning profiles and process priorities.

Sections:

1. Adjust Tuning Profiles
2. Influence Process Scheduling

## 1. Adjust Tuning Profiles

- `tuned` daemon applies tuning profiles.
- Static tuning: parameters are set at service start or profile change.
- Dynamic tuning: `tuned` watches the load and adjusts settings. Disabled by default: `dynamic_tuning = 1` in `/etc/tuned/tuned-main.conf` (`update_interval` = seconds).

```bash
dnf install tuned
systemctl enable --now tuned
```

| Profile                                  | Purpose                                         |
| ---------------------------------------- | ----------------------------------------------- |
| `balanced`                               | compromise between power saving and performance |
| `powersave`                              | maximum power saving                            |
| `throughput-performance`                 | maximum throughput                              |
| `latency-performance`                    | low latency, more power                         |
| `network-throughput` / `network-latency` | network tuning                                  |
| `virtual-guest` / `virtual-host`         | inside a VM / host for VMs                      |

- Profiles: `/usr/lib/tuned/` (do not edit) and `/etc/tuned/` (custom, wins if the name is the same). Copy a profile to `/etc/tuned/` and edit the copy.
- `tuned.conf`: `[main]` (`summary`, `include=` inherits another profile) and plug-in sections like `[sysctl]`.

| Command                         | Use                 |
| ------------------------------- | ------------------- |
| `tuned-adm active`              | current profile     |
| `tuned-adm list`                | available profiles  |
| `tuned-adm profile_info [name]` | profile details     |
| `tuned-adm profile name`        | switch profile      |
| `tuned-adm recommend`           | recommended profile |
| `tuned-adm off`                 | turn off tuning     |

- Web console: turn on administrative access, Overview, Performance profile, select, Change profile.

### Dev use

```bash
tuned-adm profile throughput-performance   # tune a server for load tests
```

## 2. Influence Process Scheduling

- The scheduler switches between processes quickly (time-slicing). Normal processes use the Completely Fair Scheduler (CFS).
- Nice value: from `-20` (highest priority) to `19` (lowest). Default `0`. A process inherits it from its parent.
- Higher nice = lower priority ("nicer" to others).
- Regular users can only increase nice, only for their own processes. Only `root` can decrease nice or change other users' processes.
- Priority matters only when the CPU is saturated.

```bash
nice sleep 60 &              # start with nice +10 (default)
nice -n 15 sleep 60 &        # start with nice 15
ps -o pid,comm,nice 2740     # show nice
renice -n 19 2740            # change nice of a running process
ps axo pid,comm,nice,cls --sort=-nice
```

- `top`: `NI` = nice, `PR` = priority (nice `-20` = PR 0, nice 19 = PR 39). Press `r` to renice.
- `CLS`: `TS` = normal, `FF` and `RR` = real-time (no nice value).

### Dev use

```bash
nice -n 19 ./heavy-build.sh   # long build that does not slow down other work
```
