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

## Making WireGuard Persistent

To ensure WireGuard starts automatically on boot and persists across system reboots, we'll create a systemd service for both the proxy server and client server.

### 1. Create the WireGuard Setup Script

#### For the Proxy Server

Create a new file `/etc/init.d/wireguard-setup` on the proxy server:

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          wireguard-setup
# Required-Start:    $network $remote_fs $syslog
# Required-Stop:     $network $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Setup WireGuard interface
# Description:       Setup WireGuard interface with IP address
### END INIT INFO

case "$1" in
  start)
    echo "Starting WireGuard..."
    modprobe wireguard
    ip link add dev wg0 type wireguard 2>/dev/null || true
    wg setconf wg0 /etc/wireguard/wg0.conf
    ip address add dev wg0 10.0.0.1/24 2>/dev/null || true
    ip link set up dev wg0
    ;;
  stop)
    echo "Stopping WireGuard..."
    ip link del dev wg0 2>/dev/null || true
    ;;
  restart)
    $0 stop
    sleep 1
    $0 start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
    ;;
esac

exit 0
```

#### For the Client Server

Create a new file `/etc/init.d/wireguard-setup` on the client server:

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          wireguard-setup
# Required-Start:    $network $remote_fs $syslog
# Required-Stop:     $network $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Setup WireGuard interface
# Description:       Setup WireGuard interface with IP address
### END INIT INFO

case "$1" in
  start)
    echo "Starting WireGuard..."
    modprobe wireguard
    ip link add dev wg0 type wireguard 2>/dev/null || true
    wg setconf wg0 /etc/wireguard/wg0.conf
    ip address add dev wg0 10.0.0.2 peer 10.0.0.1 2>/dev/null || true
    ip link set up dev wg0
    ;;
  stop)
    echo "Stopping WireGuard..."
    ip link del dev wg0 2>/dev/null || true
    ;;
  restart)
    $0 stop
    sleep 1
    $0 start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
    ;;
esac

exit 0
```

### 2. Create the Systemd Service

Create a new file `/etc/systemd/system/wireguard.service` on both servers:

```ini
[Unit]
Description=Setup WireGuard interface
After=network.target

[Service]
Type=oneshot
ExecStart=/etc/init.d/wireguard-setup start
ExecStop=/etc/init.d/wireguard-setup stop
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### 3. Enable and Start the Service

Run the following commands to enable and start the WireGuard service:

```bash
# Make the setup script executable
sudo chmod +x /etc/init.d/wireguard-setup

# Reload systemd configuration
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

# Enable and start the service
sudo systemctl enable wireguard.service
sudo systemctl start wireguard.service
```

Now WireGuard will automatically start on boot and maintain its configuration across system reboots.

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