# Chapter 6. Manage SELinux Security

Goal: protect and manage server security with SELinux.

Sections:

1. Change the SELinux Enforcement Mode
2. Control SELinux File Contexts
3. Adjust SELinux Policy with Booleans
4. Investigate and Resolve SELinux Issues

## 1. Change the SELinux Enforcement Mode

- SELinux = Mandatory Access Control (MAC) on top of normal file permissions (DAC). Anything not allowed by a rule is denied.
- Every file, process, directory and port has a context: user, role, type, level. Targeted policy uses the type (ends with `_t`).
- Example: process `httpd_t` can read files labeled `httpd_sys_content_t` (`/var/www/html`), but not `tmp_t` files (`/tmp`).
- `-Z` shows contexts: `ps axZ`, `ls -Z`.

| Mode       | Meaning                            |
| ---------- | ---------------------------------- |
| Enforcing  | policies are enforced (default)    |
| Permissive | only logs violations (for testing) |
| Disabled   | SELinux off (not recommended)      |

```bash
getenforce        # current mode
setenforce 0      # Permissive (until reboot)
setenforce 1      # Enforcing
```

- Kernel parameters at boot: `enforcing=0|1`, `selinux=0|1`. They override the config file.
- After Permissive -> Enforcing, reboot so services become confined.
- Persistent mode: `/etc/selinux/config`:

```
SELINUX=enforcing
SELINUXTYPE=targeted
```

- RHEL 9: `SELINUX=disabled` does not fully disable SELinux (it runs with no policy loaded, so everything is denied). Full disable only with `selinux=0` (`grubby --update-kernel ALL --args selinux=0`).

### Dev use

```bash
setenforce 0   # debug: is SELinux blocking my app? (then set it back to 1)
```

- If the app works in Permissive, SELinux was the cause. Fix the context instead of leaving it off.

## 2. Control SELinux File Contexts

- A new file gets its label from a labeling policy for its path. If there is no policy, it inherits the label of the parent directory.
- `cp` creates a new file and uses the context of the destination. `mv` (same file system) keeps the old context.
- `cp -p` and `cp --preserve=context` keep the context.
- After `cp` or `mv`, check with `ls -Z`.

| Command                                     | Use                                                |
| ------------------------------------------- | -------------------------------------------------- |
| `semanage fcontext -a -t TYPE 'path(/.*)?'` | add a context rule to the policy                   |
| `semanage fcontext -l` / `-l -C`            | list all rules / local rules only                  |
| `restorecon -Rv path`                       | apply the policy context (recursive, verbose)      |
| `chcon -t TYPE path`                        | change the context directly (temporary, for tests) |

- Recommended: `semanage fcontext -a`, then `restorecon`. `restorecon` also overwrites contexts set manually with `chcon`.
- `(/.*)?` = the directory itself and everything inside it.
- Packages: `policycoreutils`, `policycoreutils-python-utils`.

```bash
mkdir /virtual
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
restorecon -Rv /virtual
ls -Zd /virtual/
```

### Dev use

```bash
ls -Z /var/www/html/   # file moved from /tmp is still user_tmp_t? then: restorecon -Rv /var/www/html
```

- Common problem: you deploy files with `mv` or `rsync` and the web server gets "403". Check the context. Not from the course.

## 3. Adjust SELinux Policy with Booleans

- A Boolean turns optional behavior of an SELinux policy on or off.
- Booleans are service-specific. Docs: man pages `*_selinux` (for example `httpd_selinux(8)`), package `selinux-policy-doc`.
- Only privileged users can change them.
- Example: `httpd_enable_homedirs` lets `httpd` share user home directories (default `off`).

| Command                           | Use                                      |
| --------------------------------- | ---------------------------------------- |
| `getsebool -a` / `getsebool name` | show Booleans                            |
| `setsebool name on`               | change now (lost after reboot)           |
| `setsebool -P name on`            | change permanently                       |
| `semanage boolean -l`             | list: (current, default) and description |
| `semanage boolean -l -C`          | only Booleans changed from the default   |

```bash
getsebool httpd_enable_homedirs
setsebool -P httpd_enable_homedirs on
semanage boolean -l | grep httpd_enable_homedirs
```

### Dev use

```bash
setsebool -P httpd_can_network_connect on   # allow the web server to call other services (API, DB)
```

- Typical case: a reverse proxy or web app cannot connect to a backend because SELinux blocks it. Not from the course.

## 4. Investigate and Resolve SELinux Issues

- No policy rule for process + resource + action = the action is denied and logged.
- Most denials mean SELinux works as intended. The most common cause: wrong context on a new, copied or moved file.
- SELinux does not replace file permissions or ACLs.
- First check the `_selinux` man page (context type, Booleans), then the labels of the process and the file.
- Denials are logged as AVC in `/var/log/audit/audit.log`. The `setroubleshoot-server` package adds a summary with a UUID to `/var/log/messages`.

| Command                               | Use                                       |
| ------------------------------------- | ----------------------------------------- |
| `sealert -l UUID`                     | full report for one event                 |
| `sealert -a /var/log/audit/audit.log` | report for all events                     |
| `ausearch -m AVC -ts recent`          | search AVC events (`-m` type, `-ts` time) |

```bash
mv /root/mypage /var/www/html     # wrong context, curl returns 403
sealert -l 95f41f98-...           # advice: restorecon
restorecon -v /var/www/html/mypage
restorecon -R /var/www/html       # many files
```

- `sealert` advice has a confidence rating, but it may not fit your case. If the file is in the wrong place, move or rename it and restore the context instead of creating a new policy (`audit2allow`).
- Web console: SELinux menu shows errors and offers "Apply the solution".

### Dev use

```bash
ausearch -m AVC -ts recent   # app gets "permission denied" but file permissions are fine? check SELinux
```
