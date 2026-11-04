# Chapter 3. Analyze and Store Logs

Goal: find and read system event logs for troubleshooting.

Sections:

1. Describe System Log Architecture
2. Review Syslog Files
3. Review System Journal Entries
4. Preserve the System Journal
5. Maintain Accurate Time

## 1. Describe System Log Architecture

- RHEL 9 logging is based on the syslog protocol. Two services: `systemd-journald` and `rsyslog`.
- `systemd-journald` collects messages from the kernel, early boot, stdout/stderr of daemons and syslog. It writes them to a structured, indexed journal.
- By default the journal does **not** persist across reboots.
- `rsyslog` reads syslog messages from the journal and writes them to files in `/var/log` (persistent) or forwards them.

| Log file            | Content                                            |
| ------------------- | -------------------------------------------------- |
| `/var/log/messages` | most syslog messages (not auth, mail, cron, debug) |
| `/var/log/secure`   | security and authentication                        |
| `/var/log/maillog`  | mail server                                        |
| `/var/log/cron`     | scheduled jobs                                     |
| `/var/log/boot.log` | non-syslog console messages from boot              |

- Some apps (like Apache) write their own logs in a subdirectory of `/var/log`.

### Dev use

```bash
tail -f /var/log/messages   # follow logs live
```

## 2. Review Syslog Files

- Each syslog message has a facility (source) and a priority (severity).
- Priorities (high to low): `emerg` 0, `alert` 1, `crit` 2, `err` 3, `warning` 4, `notice` 5, `info` 6, `debug` 7.
- Common facilities: `kern`, `mail`, `daemon`, `auth`, `cron`, `authpriv`, `local0`-`local7`.
- Rules are in `/etc/rsyslog.conf` and `/etc/rsyslog.d/*.conf`.
- Rule format: `facility.priority  file`. `*` = any value, `none` = exclude.
- One message can match several rules and be saved in several files.

```
authpriv.*                                  /var/log/secure
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
local7.*                                    /var/log/boot.log
```

- `logrotate` renames old logs with a date (`messages-20220320`) and creates a new one. Runs daily, keeps about 4 weeks.

Log entry format:

```
Mar 20 20:11:48 localhost sshd[1433]: Failed password for student from 172.25.0.10 port 59344 ssh2
```

Time, host, program[PID], message. Oldest entries are at the top.

```bash
tail -f /var/log/secure                               # follow new lines
logger -p local7.notice "Log entry created on host"   # test rsyslog (default: user.notice)
```

### Dev use

```bash
grep 'Failed password' /var/log/secure   # failed SSH logins
```

## 3. Review System Journal Entries

- `systemd-journald` stores events in a structured, indexed binary journal.
- By default the journal is in `/run/log` (memory), so it is lost on reboot.
- Read it with `journalctl`. Root sees everything, regular users see less.

| Command                                                            | Use                                   |
| ------------------------------------------------------------------ | ------------------------------------- |
| `journalctl -n 5`                                                  | last 5 entries (default 10)           |
| `journalctl -f`                                                    | follow new entries (`Ctrl+C` to exit) |
| `journalctl -p err`                                                | priority `err` and higher             |
| `journalctl -u sshd.service`                                       | entries of one unit                   |
| `journalctl --since today`                                         | entries since today                   |
| `journalctl --since "2022-03-11 20:30" --until "2022-03-14 10:00"` | time range                            |
| `journalctl -o verbose`                                            | show all fields                       |
| `journalctl _SYSTEMD_UNIT=sshd.service _PID=2110`                  | filter by fields                      |

- `--since` and `--until` use `"YYYY-MM-DD hh:mm:ss"` (quotes required). Also `yesterday`, `today`, `tomorrow`, `"-1 hour"`.
- Useful fields: `_COMM`, `_EXE`, `_PID`, `_UID`, `_SYSTEMD_UNIT`.

### Dev use

```bash
journalctl -u myapp.service -p err --since "-1 hour"   # errors of my service in the last hour
```

- Quick way to debug a service that crashed. Not from the course.

## 4. Preserve the System Journal

- By default the journal is in `/run/log` (memory) and is cleared on reboot.
- Config: `/etc/systemd/journald.conf`, parameter `Storage`:

| Value        | Meaning                                                          |
| ------------ | ---------------------------------------------------------------- |
| `persistent` | store in `/var/log/journal` (survives reboot)                    |
| `volatile`   | store in `/run/log/journal` (memory only)                        |
| `auto`       | persistent if `/var/log/journal` exists, else volatile (default) |
| `none`       | no storage                                                       |

- Even a persistent journal is rotated monthly and limited (max 10% of the file system, keeps 15% free).

Enable a persistent journal:

```bash
mkdir /var/log/journal
# in /etc/systemd/journald.conf, section [Journal]: Storage=persistent
systemctl restart systemd-journald
```

Entries by boot:

```bash
journalctl --list-boots   # list boots
journalctl -b             # current boot
journalctl -b -1          # previous boot
```

### Dev use

```bash
journalctl -b -1 -p err   # errors from the previous boot (after a crash)
```

- Needs a persistent journal. Not from the course.

## 5. Maintain Accurate Time

- Accurate time is important for logs from several machines. NTP syncs the clock. In RHEL the service is `chronyd`.

| Command                                      | Use                                 |
| -------------------------------------------- | ----------------------------------- |
| `timedatectl`                                | show time, time zone, NTP status    |
| `timedatectl list-timezones`                 | list time zones                     |
| `timedatectl set-timezone America/Phoenix`   | change time zone (`UTC` also works) |
| `timedatectl set-time "YYYY-MM-DD hh:mm:ss"` | set time manually                   |
| `timedatectl set-ntp false`                  | disable NTP (`true` to enable)      |

- `tzselect` only shows the right time zone name, it does not change it.
- `set-time` fails if NTP is on: run `set-ntp false` first.

### chronyd

- Config: `/etc/chrony.conf`. Default: servers from NTP Pool Project.
- Stratum 0 = reference clock, 1 = server attached to it, 2 = client of that server.
- `server` = one stratum above, `peer` = same stratum. Use `iburst` for faster first sync.

```
server classroom.example.com iburst
```

```bash
systemctl restart chronyd
chronyc sources      # `*` in column S = current time source
chronyc sources -v   # with explanations
```

### Dev use

```bash
timedatectl set-timezone UTC   # servers usually run in UTC
```

- Same time zone everywhere makes logs easier to compare. Not from the course.
