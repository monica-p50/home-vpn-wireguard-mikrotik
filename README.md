# Home VPN: WireGuard on a MikroTik hEX

A small, working WireGuard setup for a small office: staff and admins connect from home (laptop, phone, tablet) to the office network through a MikroTik hEX router, and the same router also keeps a second tunnel to a monitoring server (Wazuh). It is documented here step by step so it can be rebuilt or extended.

Everything in this repository is a **template**. Keys, public IPs and user names are placeholders. No real key or address is stored here.

## What it does

- Remote users reach office PCs (for example Remote Desktop) and shared folders from home, over an encrypted tunnel.
- Each person or device gets one fixed address (`10.98.0.x`), so logs show who connected from where.
- The monitoring dashboard recognises these addresses and labels activity as "via home VPN".
- A second, separate tunnel (`wg-vps`, `10.99.0.x`) carries router syslog and Wazuh agent traffic to the monitoring server.

## Network plan

| Item | Value |
| --- | --- |
| Router | MikroTik hEX, RouterOS 7 |
| Home VPN interface | `wg-home`, address `10.98.0.1/24`, UDP port `51820`, MTU `1420` |
| Home VPN clients | one `/32` per device, from `10.98.0.2` upward |
| Office LAN | `192.168.88.0/24` |
| Monitoring tunnel | `wg-vps`, router `10.99.0.2`, server `10.99.0.1`, router port `13232`, keepalive 25 s |
| Internet | two ISP uplinks (failover), so the router sits behind the ISP modems |

## Diagram

```
 Laptop / phone / tablet (home)
        |  WireGuard, UDP 51820, 10.98.0.x
        v
 ISP modem (port forward UDP 51820 -> router)
        |
        v
 MikroTik hEX ----------------------------------------------+
   wg-home 10.98.0.1/24     office LAN 192.168.88.0/24      |  wg-vps 10.99.0.0/24
        |                          |                        v
        |                  Windows PCs (Wazuh agent)   Wazuh server + dashboard
        +--> allowed: LAN 192.168.88.0/24                (monitoring VPS)
```

## Quick start

1. Create the interface and peers on the router: [`docs/mikrotik-setup.md`](docs/mikrotik-setup.md)
2. Open the port on the router and on the ISP modem: [`docs/firewall-and-nat.md`](docs/firewall-and-nat.md)
3. Make the client config for Windows, phone or tablet: [`docs/clients.md`](docs/clients.md) and [`examples/client.conf.example`](examples/client.conf.example)
4. Connect the dashboard: [`docs/dashboard-integration.md`](docs/dashboard-integration.md)
5. If something fails: [`docs/troubleshooting.md`](docs/troubleshooting.md)

## Security rules I follow

- Every client has its **own key pair**. The private key never leaves the device that made it, and never goes in chat, email or this repository.
- Every peer gets a single `/32` address, so one stolen device can be removed without touching the others.
- Admins are the addresses in the firewall list `vpn-admins`. Only they get the office LAN and WinBox (from the VPN only). All other VPN users are blocked from the LAN by a final drop rule. The `Name - admin` peer comment is the label shown in logs.
- Remove a peer the same day someone leaves or loses a device.
- Only UDP 51820 is open to the internet for the VPN. WinBox is reachable only from admin VPN addresses, not from the internet.
- Keep the router and the clients updated.

More: [`docs/security.md`](docs/security.md)

## Author

Monica P. MIT licence.
