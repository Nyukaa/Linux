# RH124 - Chapter 10: Configure and Secure SSH

## 1. Access the remote command line

```bash
ssh server                      # same username as local
ssh user@server                 # other username
ssh user@server hostname        # run one command, no shell
exit                            # log out
w                               # who is logged in, and from where
```

- Host key = public key of the server. Stored in `~/.ssh/known_hosts` (user) or `/etc/ssh/ssh_known_hosts` (system).
- A line in `known_hosts` has 3 fields: host names/IPs, algorithm, key.
- `StrictHostKeyChecking yes` = abort if the key does not match (recommended).

### First connection

1. On the server, get the fingerprint:

```bash
   ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub
```

2. Compare it with the one that `ssh` shows (use another channel, like a phone call).
3. If it matches, answer `yes`.

### "REMOTE HOST IDENTIFICATION HAS CHANGED"

- Cause: the server key changed (reinstall, new IP) or someone attacks the connection.
- If you know why, remove the old key and connect again:

```bash
  ssh-keygen -R server
```

ssh user@server # зайти
ssh user@server 'uptime' # одна команда
ssh-keygen -R server # после переустановки сервера (ошибка "host identification changed")

## 2. Key-based authentication

- Key pair: **private key** (secret, only mine) + **public key** (copy to servers).
- The password is never sent over the network.

```bash
ssh-keygen                           # creates ~/.ssh/id_rsa and id_rsa.pub
ssh-keygen -f ~/.ssh/mykey           # choose a file name
ssh-copy-id -i ~/.ssh/mykey.pub user@server   # put public key on the server
ssh -i ~/.ssh/mykey user@server      # login with this key
```

- Permissions: private key `600`, public key `644`.
- WARNING: `ssh-keygen` with an existing file name overwrites the old key. Make a backup.
- Use a **passphrase** to encrypt the private key.

### ssh-agent (type the passphrase only once)

```bash
eval $(ssh-agent)        # start the agent (text login)
ssh-add                  # add the default key
ssh-add ~/.ssh/mykey     # add another key
```

### Troubleshooting

```bash
ssh -v user@server       # -vv, -vvv = more details
```

Look for: allowed methods (`publickey`, `password`), `Offering public key`, `Server accepts key`.

### Client config `~/.ssh/config`

```
Host servera
    HostName     servera.example.com
    User         usera
    IdentityFile ~/.ssh/id_rsa_servera
```

Then: `ssh servera`.
ssh-keygen -t ed25519 -C "anna@laptop" # современный тип ключа (не из курса)
ssh-copy-id user@server
ssh -vvv user@server # если что-то не работает

## 3. Secure the SSH server

Config file: `/etc/ssh/sshd_config`. After every change:

```bash
sudo systemctl reload sshd
```

| Setting                             | Meaning                                          |
| ----------------------------------- | ------------------------------------------------ |
| `PermitRootLogin no`                | root cannot log in remotely                      |
| `PermitRootLogin prohibit-password` | root can log in only with a key (RHEL 9 default) |
| `PasswordAuthentication no`         | only key-based login                             |

- Why no root login: the name `root` is known, root has full rights, and it is hard to audit who did what.
- Why no passwords: no password guessing attacks over the network.
- BEFORE disabling passwords, check that the public key is in `~/.ssh/authorized_keys` on the server. If not, I lock myself out.
- Tip: keep the current session open and test a new login in a second window.
  sudo vim /etc/ssh/sshd_config # PermitRootLogin no, PasswordAuthentication no
  sudo systemctl reload sshd
  Практический совет: не закрывай текущую ssh-сессию, пока не проверишь вход по ключу во втором окне. Если что-то пошло не так, в старой сессии можно откатить настройку. Этот совет мой, в курсе его нет.
