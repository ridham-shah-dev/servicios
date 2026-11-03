# DNS on Windows Server

**Lab:** domain `advocatspedralbes.cat`, network `192.168.200.0/24`

| Machine | Role | IP |
|---------|------|----|
| srvdnsridhams | DNS server (ns1) | 192.168.200.21 |
| www | fake web server | 192.168.200.10 |
| Windows 10 client | test client | 192.168.200.80 |

Change the names and IPs to the ones in your exam paper.

## Step 1. Name of the server

Settings > About > Rename this PC. Use **srvdns + your name + first letter of your surname** (example: `srvdnsridhams`). Restart. Check with `ipconfig /all`.

## Step 2. Static IP

`Win + R`, type `ncpa.cpl`, right click the adapter, Properties, IPv4:

| Field | Value |
|-------|-------|
| IP address | 192.168.200.21 |
| Mask | 255.255.255.0 |
| Gateway | 192.168.200.1 (only if the paper asks for it) |
| DNS | The server itself (192.168.200.21) |

Use the adapter that is on your internal network (for example *Instancia de Ethernet 0 2*), not the one that gets its IP by DHCP from Isard.

## Step 3. Install the DNS role

Server Manager > Dashboard > **Add roles and features** > Role-based installation > select your server > tick **DNS Server** > Add features > Next, Next > Install.

## Step 4. Create the forward zone

Server Manager > Tools > **DNS**. Right click **Forward Lookup Zones** > New Zone:

| Option | Value |
|--------|-------|
| Zone type | Primary zone |
| Zone name | advocatspedralbes.cat |
| Zone file | Default |
| Dynamic update | Default |

## Step 5. Add host records

Right click the zone > **New Host (A or AAAA)**. Tick **Create associated pointer (PTR) record**.

| Name | IP |
|------|----|
| ns1 | 192.168.200.21 (the DNS server) |
| www | 192.168.200.10 (a fake web server) |

> A warning appears when the reverse zone does not exist yet. This is normal. Click OK and continue. Do the reverse zone in the next step.

## Step 6. Create the reverse zone

Right click **Reverse Lookup Zones** > New Zone > Primary > **IPv4 Reverse Lookup Zone** > Network ID: `192.168.200` > defaults > Finish.

Then right click the new zone > **New Pointer (PTR)**:

| Host IP | Host name |
|---------|-----------|
| 192.168.200.21 | ns1.advocatspedralbes.cat |
| 192.168.200.10 | www.advocatspedralbes.cat |

## Step 7. Test from a Windows 10 client

Put the client in the same internal network. Set IP `192.168.200.80`, mask `255.255.255.0`, DNS `192.168.200.21`. Then in CMD:

```
ping ns1.advocatspedralbes.cat       replies from 192.168.200.21
ping www.advocatspedralbes.cat       shows 192.168.200.10 but times out
nslookup 192.168.200.21              ns1.advocatspedralbes.cat
nslookup 192.168.200.10              www.advocatspedralbes.cat
```

> The ping to `www` fails because that machine does not exist. This is **correct**. What matters is that the name resolves to `192.168.200.10`.

If even `ns1` does not answer, check that the Windows firewall on the server is not blocking ping.

## Quick fixes

| Problem | Check this |
|---------|------------|
| Client ignores my DNS | The client DNS is set to `192.168.200.21`. Run `ipconfig /flushdns` |
| Ping by IP fails | Same internal network in Isard? Firewall? Correct mask? |
| Reverse lookup shows `UnKnown` | The reverse zone and the PTR records exist and match the IPs |
| Names outside do not work | In the DNS console, right click the server > Properties > Forwarders. Add `8.8.8.8` |
