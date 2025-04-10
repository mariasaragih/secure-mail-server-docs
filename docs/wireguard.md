# WireGuard VPN Configuration

This document describes the WireGuard VPN setup between the proxy server and the client server.

## Overview

WireGuard provides a secure, encrypted tunnel between the proxy server and the client server, ensuring that all email traffic between them is protected. The client server only needs UDP port 51820 open to establish this connection, making it virtually invisible on the public internet.

## Installation

### On both servers (Proxy and Client):

```bash
# For Debian/Ubuntu
apt update
apt install wireguard

# For CentOS/RHEL
yum install epel-release
yum install wireguard-tools
```

## Key Generation

### On both servers:

```bash
# Generate private key
umask 077
wg genkey | tee /etc/wireguard/private.key | wg pubkey > /etc/wireguard/public.key
```

## Configuration Files

### Proxy Server (/etc/wireguard/wg0.conf)

```ini
[Interface]
Address = 10.0.0.1/24
PrivateKey = <proxy-server-private-key>
ListenPort = 51820

[Peer]
PublicKey = <client-server-public-key>
AllowedIPs = 10.0.0.2/32
```

### Client Server (/etc/wireguard/wg0.conf)

```ini
[Interface]
Address = 10.0.0.2/24
PrivateKey = <client-server-private-key>

[Peer]
PublicKey = <proxy-server-public-key>
Endpoint = <proxy-server-public-ip>:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 25
```

## Setting Up WireGuard

### On both servers:

```bash
# Create the WireGuard interface
ip link add dev wg0 type wireguard

# Add IP address
ip address add dev wg0 10.0.0.1/24  # On proxy server
ip address add dev wg0 10.0.0.2/24  # On client server

# Configure WireGuard using the config file
wg setconf wg0 /etc/wireguard/wg0.conf

# Bring up the interface
ip link set up dev wg0
```

## Firewall Considerations

### Proxy Server
The proxy server needs to allow incoming and outgoing traffic on UDP port 51820 for WireGuard:

```bash
sudo ufw allow 51820/udp
```

### Client Server
The client server only needs UDP port 51820 open for WireGuard. No other ports need to be exposed to the internet:

```bash
sudo ufw allow 51820/udp
```

With this minimal configuration, the client server remains invisible on the public network while still maintaining secure communication with the proxy server through the WireGuard tunnel.

## Testing the Connection

From the client server:
```bash
ping 10.0.0.1
```

From the proxy server:
```bash
ping 10.0.0.2
```

## Next Steps

**You are at Step 3 of 7 in the setup process**

Now that you have configured the secure tunnel between your servers, proceed to:

➡️ **Step 4: [Set up Mailcow](mailcow.md)** on the client server

Future steps:
5. Configure [DNS records](dns.md)
6. Set up [Haraka mail server](haraka.md)
7. Configure [Caddy web server](caddy.md) 