# Firewall and NAT

These are the rules the home VPN relies on, in the order they must appear. Order matters: the "accept" rules for admins sit **above** the final "drop" rule for everyone else on the VPN.

## Rules (RouterOS, filter table)

```
# 1. Let the VPN reach the router (WireGuard, UDP 51820)
/ip firewall filter add chain=input action=accept protocol=udp dst-port=51820 comment="WireGuard home VPN"

# 2. Admins may manage the router with WinBox, only from the VPN
/ip firewall filter add chain=input action=accept protocol=tcp dst-port=8291 src-address-list=vpn-admins in-interface=wg-home comment="WinBox from VPN admins"

# 3. Admins get the full office LAN
/ip firewall filter add chain=forward action=accept src-address-list=vpn-admins in-interface=wg-home out-interface=bridge-lan comment="VPN admins full LAN access"

# 4. Everyone else on the VPN is blocked from the office LAN
/ip firewall filter add chain=forward action=drop in-interface=wg-home out-interface=bridge-lan comment="home VPN to office LAN"
```

Rules 3 and 4 must stay in this order, and both must come before any general "accept" rule that would let VPN traffic through.

## Who is an admin

Admins are the addresses in the address list `vpn-admins`. Add one entry per admin device:

```
/ip firewall address-list add list=vpn-admins address=10.98.0.2 comment="Alice laptop"
/ip firewall address-list add list=vpn-admins address=10.98.0.3 comment="Bob laptop"
```

The word `admin` in the peer comment (`Name - admin`) is only a label that appears in the logs and on the dashboard. **The list is what grants access**, so keep the two in step.

## What each kind of user gets

| User | Reaches the router (WinBox) | Reaches office PCs |
| --- | --- | --- |
| Admin (in `vpn-admins`) | yes, TCP 8291 from the VPN only | yes, full LAN |
| Other VPN user | no | no (blocked by rule 4) |

To give one normal user access to one PC (for example Remote Desktop), add an accept rule **above rule 4**:

```
/ip firewall filter add chain=forward action=accept in-interface=wg-home out-interface=bridge-lan src-address=10.98.0.7 dst-address=192.168.88.10 protocol=tcp dst-port=3389 place-before=<rule 4 number> comment="VPN user: Remote Desktop to one PC"
```

## NAT

The home VPN needs no NAT rule of its own. The office LAN is masqueraded out of the two uplinks as usual, and a separate masquerade covers traffic to the monitoring server over `wg-vps`.

## Port forward on the ISP modems

The router has two uplinks and sits behind the ISP modems, so each modem must forward **UDP 51820** to the router. Without it nobody can connect from outside.

| Uplink | Forward on the modem |
| --- | --- |
| ISP 1 modem | UDP 51820 -> router address on that uplink |
| ISP 2 modem | UDP 51820 -> router address on that uplink |

Clients use a dynamic DNS name that follows whichever uplink is live, or the address of the primary line.

## Check

```
/ip firewall filter print where dst-port=51820 or comment~"VPN"
/ip firewall address-list print where list=vpn-admins
/ip firewall connection print where protocol=udp and dst-port=51820
```
