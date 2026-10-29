## 1. Identify automatically started processes

- `systemd` is PID 1. It starts and manages services.
- A daemon is a background process. Names usually end with `d` (`sshd`, `crond`).

| Unit type  | Used for                                          |
| ---------- | ------------------------------------------------- |
| `.service` | system services                                   |
| `.socket`  | start a service on demand when a client connects  |
| `.path`    | start a service when a file system change happens |

```bash
systemctl list-units --type=service          # loaded and active services
systemctl list-units --type=service --all    # all, even inactive
systemctl list-unit-files --type=service     # all installed unit files
systemctl status sshd.service                # detailed status
systemctl is-active sshd.service             # running now?
systemctl is-enabled sshd.service            # starts at boot?
systemctl --failed --type=service            # failed services
```

- `enabled` = starts at boot. `active (running)` = running now. These are different.
- `static` = cannot be enabled, another unit may start it.
  systemctl status nginx # что с сервисом
  systemctl is-active docker # одним словом: жив или нет
  systemctl list-units --type=service # что работает на сервере
  systemctl --failed # что упало

## 2. Control services

```bash
sudo systemctl start nginx               # start now
sudo systemctl stop nginx                # stop now
sudo systemctl restart nginx             # stop + start (PID changes)
sudo systemctl reload nginx              # reload config (PID stays)
sudo systemctl reload-or-restart nginx   # reload if possible, else restart
sudo systemctl enable nginx              # start at boot
sudo systemctl enable --now nginx        # start at boot AND start now
sudo systemctl disable --now nginx       # no autostart AND stop now
sudo systemctl mask sendmail             # block completely
sudo systemctl unmask sendmail
systemctl list-dependencies nginx        # what it needs (--reverse = what needs it)
```

- `start`/`stop` = now. `enable`/`disable` = at boot. They are independent.
- `disabled` service can still be started by hand. `masked` service cannot be started at all.
- A service can be started again by `.socket` or `.path` units even after `stop`.
  sudo systemctl restart nginx # после изменения конфига
  sudo systemctl reload nginx # без обрыва соединений
  sudo systemctl enable --now docker # включить и запустить сразу
  systemctl status myapp # что с сервисом
