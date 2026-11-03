# DNS on Linux (Debian 13, bind9)

**Lab:** domain `mired.org`, network `192.168.100.0/24`

| Machine | Role | IP |
|---------|------|----|
| asterix | DNS server (also www, mail, dns) | 192.168.100.2 |
| panoramix | router | 192.168.100.1 |
| obelix | client | 192.168.100.150 |

Change the names and IPs to the ones in your exam paper.

## How a client asks for a name

1. The client looks in its **hosts file** and its **DNS cache**.
2. If not found, it asks the DNS server.
3. If the server owns the zone, it answers. If not, it asks the **forwarder** (8.8.8.8).
4. The client gets the IP.

## Record types

| Record | Meaning |
|--------|---------|
| A | Name to IPv4 address |
| AAAA | Name to IPv6 address |
| CNAME | Alias. Another name for an existing name (not an IP) |
| NS | Which server is the name server of the domain |
| MX | Which server receives the mail |
| PTR | Reverse. IP to name |
| SOA | Info about the zone (serial, refresh, retry, expire, TTL) |

## Step 1. Static IP

Find your interface name, then set the IP. This lasts until reboot, which is enough for the practice.

```bash
ip a
sudo ip addr add 192.168.100.2/24 dev enp2s0
ip a
```

## Step 2. Install

```bash
sudo apt update
sudo apt install bind9 dnsutils
```

The package is called bind9, but the program that runs is called **named**.

## Step 3. Forwarders (for names outside your network)

```bash
sudo nano /etc/bind/named.conf.options
```

```
options {
    directory "/var/cache/bind";
    forwarders { 8.8.8.8; };
    dnssec-validation no;
};
```

> `dnssec-validation no;` is only needed if names outside do not resolve (problems with the school DNS cache).

## Step 4. Say which zone you have

```bash
sudo nano /etc/bind/named.conf.local
```

```
zone "mired.org" {
    type master;
    file "/etc/bind/db.mired.org";
};
```

## Step 5. Write the zone file

```bash
sudo nano /etc/bind/db.mired.org
```

```
$ORIGIN mired.org.
$TTL 1D
@           IN  SOA    dns admin ( 1 8H 1H 1W 1D )

@           IN  NS     asterix
@           IN  MX 10  asterix
@           IN  A      192.168.100.2

panoramix   IN  A      192.168.100.1
asterix     IN  A      192.168.100.2
obelix      IN  A      192.168.100.150

router      IN  CNAME  panoramix
www         IN  CNAME  asterix
mail        IN  CNAME  asterix
dns         IN  CNAME  asterix
```

## Step 6. Check before restarting

```bash
sudo named-checkconf
sudo named-checkzone mired.org /etc/bind/db.mired.org
```

You want to see `OK` and `loaded serial 1`. If there is an error, it shows the line number.

## Step 7. Restart and test

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
sudo ss -plunt | grep named

nslookup www.mired.org localhost
nslookup obelix.mired.org localhost
nslookup www.google.es localhost
```

The last one must say **Non-authoritative answer**. That means the forwarder works.

If a name from your zone does not work, read the log:

```bash
sudo journalctl -e -u named
```

## Step 8. Make a client use your DNS

A running server is not enough. The client must be told to use it.

| Client | How |
|--------|-----|
| Windows 10 | `Win + R`, type `ncpa.cpl`, right click the adapter, Properties, IPv4, set DNS to `192.168.100.2`. Then run `ipconfig /flushdns` |
| Linux | Check `cat /etc/resolv.conf`. Or test directly: `nslookup www.mired.org 192.168.100.2` |

Test from the client: `ping 192.168.100.2` first (the network works), then `nslookup www.mired.org`.

## Bonus. Reverse zone (IP to name)

Add this block to `/etc/bind/named.conf.local`:

```
zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.100";
};
```

Create the file `/etc/bind/db.192.168.100`:

```
$ORIGIN 100.168.192.in-addr.arpa.
$TTL 1D
@     IN  SOA  dns.mired.org. admin.mired.org. ( 1 8H 1H 1W 1D )
@     IN  NS   asterix.mired.org.
2     IN  PTR  asterix.mired.org.
1     IN  PTR  panoramix.mired.org.
150   IN  PTR  obelix.mired.org.
```

```bash
sudo systemctl restart bind9
nslookup 192.168.100.2 localhost
```

## Mistakes that break the zone file

| Mistake | Example | Fix |
|---------|---------|-----|
| Typo in a keyword | `tpye master`, `zona`, `CMANE` | Must be `type`, `zone`, `CNAME`. `named-checkconf` shows it |
| Missing dot at the end of a full name | `dns.mired.org` | Write `dns.mired.org.` (with the last dot) or just `dns` |
| CNAME pointing to an IP | `www CNAME 192.168.100.2` | CNAME points to a name. Use an A record for IPs |
| Edited the zone but nothing changed | Same old answer | Raise the serial number (1 to 2) and restart |
| Wrong file permissions | named cannot read the file | `sudo chown root:bind /etc/bind/db.mired.org` |

## Quick fixes

| Problem | Check this |
|---------|------------|
| Zone does not load | `named-checkconf` and `named-checkzone`. Look for typos and missing dots |
| Names outside do not work | `forwarders { 8.8.8.8; };` is set. Try `dnssec-validation no;` |
| Client ignores my DNS | The client DNS is set to the server IP. Run `ipconfig /flushdns` |
| Ping by IP fails | Same network? Firewall? Correct mask? Fix the network first, then DNS |
