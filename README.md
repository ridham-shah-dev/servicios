# Servicios

Short guides to install and test network services. Each service has one guide for **Linux (Debian 13)** and one for **Windows Server**.

Module: M07 Serveis de Xarxa (CFGM SMX, Institut Pedralbes)

| Service | Linux | Windows |
|---------|-------|---------|
| DNS | [dns/linux](dns/linux/README.md) | [dns/win](dns/win/README.md) |
| DHCP | [dhcp/linux](dhcp/linux/README.md) | [dhcp/win](dhcp/win/README.md) |

## The 2 ideas

| | DNS | DHCP |
|---|-----|------|
| Job | Turns a name into an IP (`www.mired.org` to `192.168.100.150`). Like a phone book. | Gives IP, mask, gateway and DNS to clients automatically. |
| Port | 53 (UDP, sometimes TCP) | UDP 67 (server) and 68 (client) |
| Linux software | bind9 (the service is called `named`) | isc-dhcp-server, or dnsmasq for both |
| Windows role | DNS Server | DHCP Server |

> **Golden rule:** the server must have a **static IP**. Do this first. Check it with `ip a` (Linux) or `ipconfig /all` (Windows).

## Same job, both systems

| I want to... | Linux | Windows |
|--------------|-------|---------|
| See my IP | `ip a` | `ipconfig /all` |
| Ask a name | `nslookup name server` or `dig name` | `nslookup name server` |
| Renew the DHCP IP | `sudo dhclient -r` then `sudo dhclient -v` | `ipconfig /release` then `ipconfig /renew` |
| Clear the DNS cache | `sudo systemd-resolve --flush-cache` | `ipconfig /flushdns` |
| Service status | `systemctl status bind9` | Server Manager > Tools > DNS or DHCP |
| Hosts file | `/etc/hosts` | `C:\Windows\System32\drivers\etc\hosts` |

## Before you hand in a practice

1. Screenshot of the config files.
2. Screenshot of the check (`named-checkzone` OK, or the DNS console).
3. Screenshot of the client test (`nslookup`, `ping`, `ipconfig /all`).
4. Your name and the date in the document.
