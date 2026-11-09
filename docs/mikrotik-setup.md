# MikroTik setup (RouterOS 7)

Run these in Winbox (New Terminal) or SSH. Replace every `<...>` placeholder.

## 1. Create the interface

```
/interface wireguard add name=wg-home listen-port=51820 mtu=1420 comment="home VPN"
/ip address add address=10.98.0.1/24 interface=wg-home comment="home VPN"
```

Show the router's **public** key. Clients need it. The private key is never shown by a normal `print`, and you should not use `show-sensitive`.

```
/interface wireguard print
```

## 2. Add one peer per device

One peer = one device = one fixed address. Write the person and the role in the comment, in the form `Name - admin` or just `Name`. For an admin, also add the address to the `vpn-admins` list (see the firewall page): the list grants access, the comment is only a label.

```
/interface wireguard peers add interface=wg-home public-key="<CLIENT_PUBLIC_KEY>" allowed-address=10.98.0.2/32 comment="Alice - admin"
/interface wireguard peers add interface=wg-home public-key="<CLIENT_PUBLIC_KEY>" allowed-address=10.98.0.3/32 comment="Bob"
```

Keep a simple address list so addresses are never reused:

| Address | Person / device | Role |
| --- | --- | --- |
| 10.98.0.2 | Alice, laptop | admin |
| 10.98.0.3 | Bob, laptop | admin |
| 10.98.0.4 | Carol, laptop | admin |
| 10.98.0.5 | Grace, laptop | admin |
| 10.98.0.6 | Frank, laptop | user |
| 10.98.0.7 | Erin, laptop | user |
| 10.98.0.8 | Dave, laptop | user |
| 10.98.0.9 | Heidi, laptop | user |
| 10.98.0.10 | Grace, tablet | admin |

Clients are not given an endpoint on the router. They connect in, and the router learns where they are.

## 3. Check

```
/interface wireguard peers print
/interface wireguard print
```

A connected peer shows a recent "last handshake". The interface shows `R` (running).

## 4. The monitoring tunnel (second interface)

```
/interface wireguard add name=wg-vps listen-port=13232 mtu=1420 comment="tunnel to monitoring server"
/ip address add address=10.99.0.2/24 interface=wg-vps
/interface wireguard peers add interface=wg-vps public-key="<SERVER_PUBLIC_KEY>" endpoint-address=<SERVER_PUBLIC_IP> endpoint-port=51820 allowed-address=10.99.0.1/32 persistent-keepalive=25s comment="monitoring server"
/ip firewall nat add chain=srcnat action=masquerade out-interface=wg-vps comment="LAN to monitoring server via wg"
```

The keepalive keeps the tunnel open through the ISP's NAT.

## 5. Back up the config

```
/export file=router-config-<date>
```

Do **not** publish an `export`: it can contain secrets and your addresses.
