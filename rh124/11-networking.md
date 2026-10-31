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
