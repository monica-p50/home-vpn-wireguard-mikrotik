# Clients (Windows, phone, tablet)

Each device makes its own key pair. The **private key stays on the device**.

## Windows or Mac

1. Install the WireGuard app from wireguard.com.
2. **Add Tunnel > Add empty tunnel.** The app creates a key pair and shows the **public key**. Copy only that.
3. Give the public key to the admin, who adds the peer on the router with the next free address.
4. Fill in the tunnel text like [`examples/client.conf.example`](../examples/client.conf.example) and save.
5. Press **Activate**.

## Android, iPhone, iPad

1. Install the WireGuard app.
2. **+ > Create from scratch.** Generate a key pair in the app and copy the public key to the admin.
3. Enter the same values as in the example file, then switch the tunnel on.

## Split tunnel

The example sends only office traffic (`10.98.0.0/24` and `192.168.88.0/24`) through the VPN. Everything else uses the home internet normally, which is faster and keeps personal browsing out of the office router.
