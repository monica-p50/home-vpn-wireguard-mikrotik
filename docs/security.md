# Security notes

| Risk | What I do |
| --- | --- |
| A stolen laptop or phone | Remove its peer on the router at once. Other devices keep working. |
| A shared key | Never share keys. One device = one key pair. |
| Open port scanning | The only port open for the VPN is UDP 51820. WireGuard does not answer packets without a valid key. |
| Too much access | `/32` per peer. Admins are in the `vpn-admins` list; every other VPN user is dropped from the office LAN. |
| Key leaks in chat or Git | Private keys and `export` files are never posted anywhere. `.gitignore` blocks `*.conf` and key files. |
| Old accounts | Review the peer list once a month and remove unused peers. |
| Outdated software | Update RouterOS and the WireGuard clients regularly. Keep a backup before updating. |
| Visibility | Router and PC logs go to Wazuh, so connections and file activity can be reviewed. |

If a private key is ever pasted somewhere public, treat it as lost: make a new key pair on that device and replace the peer on the router.
