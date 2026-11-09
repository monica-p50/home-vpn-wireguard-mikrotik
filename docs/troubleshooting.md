# Troubleshooting

| Symptom | Check |
| --- | --- |
| Client says connected but nothing works | The peer's `allowed-address` on the router must be the exact address in the client's `Address`. Check the clock and keys too. |
| No handshake at all | The ISP modem is not forwarding UDP 51820, or the router firewall input rule is missing, or the endpoint name points at the wrong uplink. |
| Handshake works, office PCs unreachable | A forward firewall rule is missing or too strict, or the PC's own firewall blocks Remote Desktop from `10.98.0.x`. |
| Works on one ISP but not the other | Forward the port on both modems. Use a dynamic DNS name that follows the live line. |
| Drops after a few minutes | Set `PersistentKeepalive = 25` on the client. |
| Slow or big files fail | Try MTU 1380 on the client. |
| Two devices fight | Two devices use the same key or address. Give each its own. |

Useful commands on the router:

```
/interface wireguard peers print detail
/log print where message~"wireguard"
/tool torch interface=wg-home
```
