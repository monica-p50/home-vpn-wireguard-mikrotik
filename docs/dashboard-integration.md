# Dashboard integration

The monitoring dashboard (see the SOC LAB dashboard repository) reads Wazuh alerts and uses the home VPN in three ways.

1. **"Via home VPN" label.** A remote logon or Remote Desktop session whose source address is inside `10.98.0.0/24` is labelled "via home VPN" instead of looking like an unknown outside address.
2. **Who it was.** The dashboard maps each VPN address to a person (setting `vpnPeers`, one line per address, for example `10.98.0.2 Alice`), so File activity shows "Home VPN - Alice".
3. **Connect and disconnect.** The router logs when a WireGuard peer connects or disconnects. The dashboard shows these as "VPN user connected / disconnected", and the role in the peer comment (`Name - admin`) is carried along.

To keep this working:

- Keep the peer comment format `Name - admin` or `Name`.
- Keep VPN addresses unique, and update `vpnPeers` when you add or remove a peer.
- Keep the Data Availability page open for a week after changes. It shows whether the VPN lines are still arriving.
