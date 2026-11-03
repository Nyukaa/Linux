# Chapter 2. Schedule Future Tasks

Goal: schedule tasks to run at a specific time and date.

## 1. Schedule a Deferred User Job

- A deferred job runs once at a future time (for example, a long maintenance task at night).
- Tool: `at`. Installed and enabled by default.
- Package `at` gives the `atd` daemon and the `at` and `atq` commands.
- Any user can queue jobs.
- `atd` has 26 queues, `a` to `z`. Later letters = lower priority (higher nice value).

### Create a job

- `at TIMESPEC` starts a new job, commands are read from STDIN.
- Finish input with `Ctrl+D` on an empty line.
- Use redirect for complex commands: `at now +5min < myscript`.

### TIMESPEC

- Natural time: `02:00pm`, `15:59`, `midnight`, `teatime` (16:00).
- Order: time first, then date (or days ahead).
- Date only: time = current time.
- Time only: runs when the time next matches (today or tomorrow).
- Case does not matter.

```bash
date                    # Wed May 18 21:01:18 CDT 2022
at 21:03 < myscript     # job 3 at Wed May 18 21:03:00 2022
at 21:00 < myscript     # job 4 at Thu May 19 21:00:00 2022 (time passed, so tomorrow)
```

```
now +5min
teatime tomorrow
noon +4 days
5pm august 3 2021
```

More examples: `/usr/share/doc/at/timespec`.

### Inspect and remove jobs

```bash
atq                     # same as: at -l (pending jobs of the current user)
at -c 28                # job environment and the commands to run
atrm 28                 # remove a job
```

```
28  Mon May 16 05:13:00 2022 a user
```

| Field                      | Meaning                               |
| -------------------------- | ------------------------------------- |
| `28`                       | unique job number                     |
| `Mon May 16 05:13:00 2022` | execution date and time               |
| `a`                        | queue (default `a`)                   |
| `user`                     | job owner (the job runs as this user) |

- Unprivileged users see and manage only their own jobs. `root` sees all jobs.
- Use case: queue a safety job that resets a firewall in 10 minutes, then `atrm` it if the new config works.

### Dev use

```bash
at now +10min < reset-config.sh   # safety job: rollback in 10 minutes
atrm 5                            # config works, cancel the rollback
```

- Good for risky changes on a remote server. For repeating tasks use cron (next section).
- The example is not from the course.

References: `at(1)`, `atd(8)`, `/usr/share/doc/at/timespec`.

## 2. Schedule Recurring User Jobs

Goal: run commands on a repeating schedule with a user's crontab file.

- Daemon: `crond`, enabled and started by default.
- It reads one file per user and a set of system-wide files.
- If a job has no output redirection, `crond` emails the output and errors to the job owner.

### crontab command

| Command            | Use                                                                         |
| ------------------ | --------------------------------------------------------------------------- |
| `crontab -l`       | list jobs of the current user                                               |
| `crontab -r`       | remove all jobs of the current user                                         |
| `crontab -e`       | edit jobs of the current user                                               |
| `crontab filename` | remove all jobs and replace them with jobs from the file (stdin if no file) |

- A privileged user can use `-u` to manage jobs of another user.
- `crontab` is never used for system jobs. Do not use it as root (personal root jobs can be exploited).

### Job format

- `crontab -e` opens `vim` unless `EDITOR` is set.
- One job per line. Allowed: empty lines, comments (`#`), variables (`NAME=value`, apply to all lines below).
- Common variables: `SHELL` (shell for the lines below), `MAILTO` (who gets the email).
- Sending email needs a local mail server or an SMTP relay.

Fields in order: Minutes, Hours, Day of month, Month, Day of week, Command.

```
15 12 11 * Fri command    # 11th of every month and every Friday, 12:15
```

| Syntax       | Meaning                                       |
| ------------ | --------------------------------------------- |
| `*`          | every possible value                          |
| number       | minutes, hours, date or day of week           |
| `x-y`        | range (includes x and y)                      |
| `x,y`        | list (can include ranges: `5,10-13,17`)       |
| `*/x`        | interval (`*/7` in Minutes = every 7 minutes) |
| `Jan`, `Mon` | 3-letter English names for months and days    |

- Day of week: `0` and `7` = Sunday, `1` = Monday, `2` = Tuesday, and so on.
- In the Command field an unescaped `%` is treated as a newline. Everything after it goes to the command as stdin.

### Examples

```
0 9 3 2 *        /usr/local/bin/yearly_backup   # 09:00 on 3 February, every year
*/5 9-16 * Jul 5 echo "Chime"                    # every 5 min, 09:00-16:55, Fridays in July
58 23 * * 1-5    /usr/local/bin/daily_report    # Mon-Fri, 23:58
0 9 * * 1-5      mutt -s "Checking in" developer@example.com % Hi there, just checking in.
```

- Hours `9-16` match all minutes of those hours. Last run is 16:55.
- Cron jobs have no controlling terminal, so `crond` buffers the output and emails it. For system jobs the email goes to `root`.

### Dev use

```bash
0 3 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1   # every night at 03:00, output to a log
```

- Without redirection, output goes to email and is lost if mail is not configured.
- Cron has a minimal environment, so use absolute paths in scripts.
- The example is not from the course.

References: `crond(8)`, `crontab(1)`, `crontab(5)`.

## 3. Schedule Recurring System Jobs

Goal: run commands on a repeating schedule with the system crontab file, directories, anacron and systemd timers.

### System crontab

- Run recurring system jobs from system accounts, not user accounts.
- Use system-wide crontab files, not the `crontab` command.
- Same format as a user crontab, but with an extra field before the command: the user who runs it.

```
# minute hour day-of-month month day-of-week user-name command
*  *  *  *  *  user-name  command to be executed
```

- System jobs are defined in `/etc/crontab` and in files in `/etc/cron.d/`.
- Create custom crontab files in `/etc/cron.d/`, so a package update does not overwrite `/etc/crontab`.
- Packages also put their crontab files in `/etc/cron.d/`. Admins group related jobs in one file there.

### Script directories

- `/etc/cron.hourly/`, `/etc/cron.daily/`, `/etc/cron.weekly/`, `/etc/cron.monthly/`.
- They contain executable shell scripts, not crontab files.
- A script must be executable: `chmod +x script_name`.

### Anacron

- `anacron` runs daily, weekly and monthly jobs from `/etc/anacrontab` with the `run-parts` script.
- Jobs are not skipped if the system was off or hibernating. They run when the system is ready (after the optional delay).
- Files in `/var/spool/anacron/` track the jobs. Their timestamp = last run time.
- `/etc/anacrontab` has four fields per line:

| Field            | Meaning                                                              |
| ---------------- | -------------------------------------------------------------------- |
| Period in days   | interval in days: integer or macro (`@daily` = `1`, `@weekly` = `7`) |
| Delay in minutes | how long to wait before the job starts                               |
| Job identifier   | unique job name in log messages                                      |
| Command          | command to run                                                       |

- `START_HOURS_RANGE` sets the time range when jobs may start. Outside the range the job waits until the next day.

### Systemd timer

- A timer unit activates another unit (for example, a service) with the same name.
- Timer events are logged in the system journal.
- Example: `sysstat-collect.timer` collects statistics every 10 minutes.

```ini
[Unit]
Description=Run system activity accounting tool every 10 minutes

[Timer]
OnCalendar=*:00/10

[Install]
WantedBy=sysstat.service
```

- `OnCalendar=*:00/10` = every 10 minutes.
- `OnCalendar=2022-04-* 12:35,37,39:16` = 12:35:16, 12:37:16, 12:39:16 every day in April 2022.
- `OnUnitActiveSec=15min` = 15 minutes after the last activation.

### Change a timer unit

1. Do not edit files in `/usr/lib/systemd/system/`.
2. Copy the file to `/etc/systemd/system/` and edit the copy. If both directories have a file with the same name, the one in `/etc/systemd/system/` is used.
3. Reload the configuration.
4. Enable and start the timer.

```bash
systemctl daemon-reload
systemctl enable --now <unitname>.timer
```

### Dev use

```bash
# /etc/cron.d/app-cleanup
30 2 * * * appuser /opt/app/cleanup.sh >> /var/log/cleanup.log 2>&1
```

- A service user runs the job (not root), and a separate file in `/etc/cron.d/` is safe from package updates.
- For new jobs a systemd timer is often easier: events are in the journal.
- The example is not from the course.

## 4. Manage Temporary Files

- `systemd-tmpfiles` creates, cleans and secures temporary files and directories.
- `/run` is in memory (cleaned on reboot). `/tmp` is on disk and needs cleaning.
- At boot: `systemd-tmpfiles --create --remove`. Cleaning: `systemd-tmpfiles --clean`.
- `systemd-tmpfiles-clean.timer` runs the clean service: `OnBootSec=15min`, then `OnUnitActiveSec=1d`.
- After changing a timer: `systemctl daemon-reload`.

Config line: `Type Path Mode UID GID Age Argument`

| Type | Action                                                  |
| ---- | ------------------------------------------------------- |
| `d`  | create directory if missing                             |
| `D`  | create directory, `--clean` removes old files           |
| `L`  | create symlink                                          |
| `Z`  | restore SELinux context, permissions, owner (recursive) |

```
D /home/student 0700 student student 1d
```

Config locations (first wins if same file name): `/etc/tmpfiles.d/` > `/run/tmpfiles.d/` > `/usr/lib/tmpfiles.d/`.
Do not edit `/usr/lib/tmpfiles.d/`. Copy the file to `/etc/tmpfiles.d/` and edit the copy.

### Dev use

```
# /etc/tmpfiles.d/app.conf
d /tmp/uploads 0755 appuser appuser 7d
```

- Deletes uploads older than 7 days. Not from the course.
