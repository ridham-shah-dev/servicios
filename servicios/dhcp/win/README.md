# DHCP on Windows Server

**Lab:** network `192.168.100.0/24`

| Machine | Role | IP |
|---------|------|----|
| Windows Server | DHCP server | 192.168.100.2 |
| Router | gateway | 192.168.100.1 |
| Clients | get IPs from `.100` to `.140` | automatic |
| obelix | reservation | 192.168.100.150 |

Change the names and IPs to the ones in your exam paper.

## Step 1. Static IP on the server

`Win + R`, type `ncpa.cpl`, right click the adapter, Properties, IPv4. Set a fixed IP, mask, gateway and DNS (example: `192.168.100.2` / `255.255.255.0`).

## Step 2. Install the DHCP role

Server Manager > **Add roles and features** > Role-based > your server > tick **DHCP Server** > Add features > Install.

When it ends, click the yellow flag at the top > **Complete DHCP configuration** > Commit > Close.

## Step 3. Create the scope

Server Manager > Tools > **DHCP**. Open your server, right click **IPv4** > **New Scope**:

| Option | Value |
|--------|-------|
| Name | Scope_Clients |
| IP range | Start `192.168.100.100`, End `192.168.100.140`, mask `255.255.255.0` (length 24) |
| Exclusions | Optional. IPs that DHCP must never give |
| Lease duration | Default is 8 days. Use 1 hour for tests |
| Configure options now | **Yes** |
| Router (default gateway) | `192.168.100.1`, then Add |
| Domain name and DNS | Parent domain `mired.org`. DNS server `192.168.100.2`, then Add |
| WINS | Skip (Next) |
| Activate scope now | **Yes** |

## Step 4. Reservation (fixed IP for one client)

Open the scope > right click **Reservations** > **New Reservation**:

| Option | Value |
|--------|-------|
| Reservation name | obelix |
| IP address | 192.168.100.150 |
| MAC address | The client MAC **without dashes**, for example `5254001A2B3C` |
| Supported types | Both |

## Step 5. Test on the client

Set the client adapter to **Obtain an IP address automatically**, then:

```
ipconfig /release
ipconfig /renew
ipconfig /all
```

Look at IPv4 address, default gateway, DNS server and DHCP server.

On the server, open the scope > **Address Leases** to see the client. A reserved client shows as a reservation.

## Not working?

If the client gets a wrong IP (for example `169.254.x.x` or an Isard IP), check these:

- The scope is **activated** (green arrow, not red).
- The client and the server are on the **same internal network** in Isard.
- No other DHCP server is in that network.

## Quick fixes

| Problem | Check this |
|---------|------------|
| Client gets no IP | Server has a static IP. Scope is active. Same network. Only one DHCP server |
| Reservation ignored | The MAC is correct and written without dashes. The reserved IP is outside the range |
| Client has no DNS | The scope options include the DNS server (option 006) |
