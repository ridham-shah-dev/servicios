# DHCP on Linux (Debian 13, isc-dhcp-server)

**Lab:** network `192.168.100.0/24`

| Machine | Role | IP |
|---------|------|----|
| DHCP server | gives IPs | 192.168.100.2 |
| Router | gateway | 192.168.100.1 |
| Clients | get IPs from `.100` to `.140` | automatic |
| obelix | reservation | 192.168.100.150 |

Change the names and IPs to the ones in your exam paper.

## Step 1. Static IP on the server

```bash
ip a
sudo ip addr add 192.168.100.2/24 dev enp2s0
```

## Step 2. Install

```bash
sudo apt update
sudo apt install isc-dhcp-server
```

It may show an error at the end because it is not configured yet. This is normal.

## Step 3. Tell it which network card to use

```bash
sudo nano /etc/default/isc-dhcp-server
```

```
INTERFACESv4="enp2s0"
```

## Step 4. Configure the scope

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

```
option domain-name "mired.org";
option domain-name-servers 192.168.100.2;
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.100 192.168.100.140;
    option routers 192.168.100.1;
}

# Reservation: this MAC always gets this IP
host obelix {
    hardware ethernet 52:54:00:aa:bb:cc;
    fixed-address 192.168.100.150;
}
```

- `option domain-name-servers` is how DHCP and DNS work together. The client receives the IP of your DNS server automatically.
- Get the client MAC with `ipconfig /all` (Physical Address). Windows shows dashes, this file needs **colons**.
- Keep the reserved IP **outside** the range.

## Step 5. Check, start, and read the leases

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf     # test the config, no output = good
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
cat /var/lib/dhcp/dhcpd.leases              # who got which IP
```

## Step 6. Test on the clients

Set the client adapter to **Obtain an IP address automatically** first.

| Client | Commands |
|--------|----------|
| Windows | `ipconfig /release`, then `ipconfig /renew`, then `ipconfig /all` (look at IP, gateway, DNS, DHCP server) |
| Linux | `sudo dhclient -r`, then `sudo dhclient -v`, then `ip a` |

## Faster option. dnsmasq (DNS and DHCP in one)

```bash
sudo apt install dnsmasq
sudo nano /etc/dnsmasq.conf
```

```
interface=enp2s0
domain=mired.org
dhcp-range=192.168.100.100,192.168.100.140,12h
dhcp-option=option:router,192.168.100.1
dhcp-option=option:dns-server,192.168.100.2
dhcp-host=52:54:00:aa:bb:cc,192.168.100.150
```

```bash
sudo systemctl restart dnsmasq
```

## Quick fixes

| Problem | Check this |
|---------|------------|
| DHCP does not start | Run `sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf`. Check the interface name in `/etc/default/isc-dhcp-server` |
| Client gets no IP | Server has a static IP. Same network. Only one DHCP server in the network |
| Reservation ignored | The MAC is correct and written with colons. The reserved IP is outside the range |
