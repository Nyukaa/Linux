# Chapter 11. Manage Network Security

Goal: control network connections to services with the firewall and SELinux rules.

## 1. Manage Server Firewalls

- RHEL 9 firewall core: `nftables` (replaced `iptables`). `firewalld` is the recommended front end. Tool: `firewall-cmd`.
- Traffic is sorted into zones. Zone choice order: source address, then interface, then the default zone (`public`).
- Each zone has open ports and services. Everything else incoming is rejected (except `trusted`).

| Zone               | Default                                                      |
| ------------------ | ------------------------------------------------------------ |
| `trusted`          | allow all incoming                                           |
| `home`, `internal` | `ssh`, `mdns`, `ipp-client`, `samba-client`, `dhcpv6-client` |
| `work`             | `ssh`, `ipp-client`, `dhcpv6-client`                         |
| `public`           | `ssh`, `dhcpv6-client` (default for new interfaces)          |
| `external`         | `ssh`, masquerading                                          |
| `dmz`              | `ssh`                                                        |
| `block` / `drop`   | reject / silently drop everything incoming                   |

- Predefined services (`ssh`, `cockpit`, `mysql`, `nfs`...): `firewall-cmd --get-services`. Or open a port: `PORT/PROTOCOL`.

| Command                                          | Use                                               |
| ------------------------------------------------ | ------------------------------------------------- |
| `--get-default-zone` / `--set-default-zone=ZONE` | show / set the default zone                       |
| `--get-active-zones`                             | zones in use                                      |
| `--list-all [--zone=ZONE]`                       | everything about a zone                           |
| `--add-service=S` / `--remove-service=S`         | allow / remove a service                          |
| `--add-port=PORT/PROTO` / `--remove-port=...`    | allow / remove a port                             |
| `--add-source=CIDR`                              | send traffic from an address or network to a zone |
| `--add-interface=IF` / `--change-interface=IF`   | bind an interface to a zone                       |
| `--reload`                                       | drop runtime config, apply permanent config       |

- No `--zone=` = default zone. Without `--permanent` the change is runtime only (lost on reload or reboot). With `--permanent` you must run `--reload`.

```bash
firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24
firewall-cmd --permanent --zone=internal --add-service=mysql
firewall-cmd --reload
firewall-cmd --list-all --zone=internal
```

- Web console: Networking, Firewall, Edit rules and zones, Add services (needs administrative access).

### Dev use

```bash
firewall-cmd --permanent --add-port=3000/tcp && firewall-cmd --reload   # open a port for a Node app
```

- Typical: the app works on localhost but not from outside. Not from the course.

## 2. Control SELinux Port Labeling

- SELinux also labels network ports. A service can listen on a port only if its policy allows binding to the port type (`ssh_port_t` = 22/tcp, `http_port_t` = 80/tcp, 443/tcp).
- A service on a nonstandard port without the right label can be blocked. Fix: label the port.
- A port can have only one type.

| Command                                     | Use                                                     |
| ------------------------------------------- | ------------------------------------------------------- |
| `semanage port -l`                          | list port labels (add `\| grep ftp` or `\| grep -w 70`) |
| `semanage port -a -t TYPE -p tcp\|udp PORT` | add a label                                             |
| `semanage port -d -t TYPE -p tcp\|udp PORT` | delete a label                                          |
| `semanage port -m -t TYPE -p tcp\|udp PORT` | modify the type of a port                               |
| `semanage port -l -C`                       | local changes only                                      |

```bash
semanage port -a -t gopher_port_t -p tcp 71
semanage port -m -t http_port_t -p tcp 71
semanage port -l -C
```

- Default port labels from the policy cannot be changed with `semanage`.
- Docs: `*_selinux` man pages: `dnf -y install selinux-policy-doc`, then `man -k _selinux`.

### Dev use

```bash
semanage port -a -t http_port_t -p tcp 3000   # web server or proxy must listen on a custom port
```

- If your service fails to bind to a custom port and the firewall is fine, check SELinux. Not from the course.
