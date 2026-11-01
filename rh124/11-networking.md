# RH124 - Chapter 11: Manage Networking

## 1. Networking concepts

| TCP/IP layer | Examples                            |
| ------------ | ----------------------------------- |
| Application  | SSH, HTTPS, FTP, SMTP               |
| Transport    | TCP (reliable), UDP (no connection) |
| Internet     | IPv4, IPv6, routers                 |
| Link         | Ethernet, Wi-Fi, MAC address        |

- Socket = IP address + port. Well-known ports: `/etc/services`.
- Interface names: `en` Ethernet, `wl` WLAN, `ww` WWAN (e.g. `ens3`, `enp0s1f0`).

### IPv4

- 32 bits. The netmask (`/24` = `255.255.255.0`) splits network and host part.
- Network address = host bits all 0. Broadcast = host bits all 1.
- `/24` = 254 hosts, `/16` = 65534 hosts, `/8` = 16,777,214 hosts.
- Routing: the longest matching prefix wins. No match = default route `0.0.0.0/0` (gateway).
- Config: DHCP (automatic) or static (config files).

### IPv6

- 128 bits. Shorten: drop leading zeros, one `::` for a run of zero groups.
- With a port: `[2001:db8::1]:80`.

| Address     | Meaning                           |
| ----------- | --------------------------------- |
| `::1`       | localhost                         |
| `::`        | all addresses (like `0.0.0.0`)    |
| `fe80::/10` | link-local (local link only)      |
| `ff00::/8`  | multicast (IPv6 has no broadcast) |

- Auto config: SLAAC (router gives the prefix) or DHCPv6.

### Names

- `/etc/hosts`: static names, edited by hand on each machine.
- DNS: servers set in `/etc/resolv.conf` (or from DHCP).

```bash
ip addr show          # addresses
ip -br addr show      # short list
```

- A service on `127.0.0.1` is not reachable from outside. `0.0.0.0` = all addresses.
  ip -br addr show # мои адреса кратко
  ip route # таблица маршрутов

## 2. Validate the network

```bash
ip link show             # interfaces and MAC addresses
ip addr show ens3        # IP addresses (inet = IPv4, inet6 = IPv6)
ip -br addr              # short list
ip -s link show ens3     # RX/TX statistics (errors, dropped)
ip route                 # routing table (IPv4)
ip -6 route              # routing table (IPv6)
ping -c3 192.0.2.254     # test connection, 3 packets
ping6 ff02::1%ens3       # IPv6 link-local needs %interface
tracepath host           # path through routers
ss -tlnp                 # listening TCP sockets with process names
```

| `ss` option | Meaning            |
| ----------- | ------------------ |
| `-t` / `-u` | TCP / UDP          |
| `-l`        | listening only     |
| `-a`        | all sockets        |
| `-n`        | numbers, not names |
| `-p`        | show the process   |

- `ss` replaces `netstat`.
- `*:22` or `[::]:22` = listens on all addresses. `127.0.0.1:25` = only local.
- `default via ...` in `ip route` is the gateway.
- High RX/TX errors or dropped packets = network problem.
  ip -br addr # мои адреса кратко
  ping -c3 example.com # есть ли связь
  ss -tlnp # кто слушает какой порт и какой процесс, самая полезная из них
  ip route # куда идёт трафик
