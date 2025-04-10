# Firewall Configuration

This document describes the firewall configuration required for the secure email server architecture.

## Overview

A properly configured firewall is essential for securing your email server. This guide covers firewall configuration for both the proxy server and client server:

1. **Proxy Server Firewall**: Controls which ports are accessible from the internet and configures port forwarding for specific mail-related services
2. **Client Server Firewall**: Minimal configuration that only allows WireGuard VPN communication

## IONOS VPS Firewall Configuration

If you're using IONOS as your VPS provider for the proxy server (as recommended in the testing environment), you'll need to configure their provider-level firewall in addition to the system firewall (UFW).

### Opening Required Ports in IONOS Control Panel

IONOS provides a web-based firewall configuration interface that you must use to open ports before they can be accessed, even if they are allowed in your system firewall.

1. Login to your IONOS control panel
2. Navigate to the "Server & Cloud" section
3. Select your VPS
4. Go to "Network" → "Firewall Rules"
5. Create rules to open the following ports:
   - 22/TCP (SSH)
   - 80/TCP (HTTP)
   - 443/TCP (HTTPS)
   - 51820/UDP (WireGuard)
   - 587/TCP (SMTP Submission)
   - 993/TCP (IMAPS)

### Special Case: Port 25 (SMTP)

IONOS blocks port 25 by default as an anti-spam measure, and this port cannot be opened directly through the control panel. To open port 25:

1. Contact IONOS customer support by phone (available 24/7)
2. Request to open port 25 for your VPS
3. Explain that you're setting up a mail server
4. They will typically open the port without issues after verifying your account

This extra step helps prevent the VPS from being used for spam but may cause a slight delay in your setup process.

## Proxy Server Firewall Configuration

### UFW Installation

UFW is usually pre-installed on most Ubuntu/Debian systems. If not, install it:

```bash
sudo apt update
sudo apt install ufw
```

### Enable IP Forwarding

Before configuring the firewall, you must enable IP forwarding at the system level to allow the server to forward packets between interfaces:

```bash
# Edit sysctl.conf
sudo nano /etc/sysctl.conf

# Add or uncomment the following line
net.ipv4.ip_forward=1

# Apply the changes without rebooting
sudo sysctl -p
```

This enables IP forwarding permanently across reboots. You can verify it's enabled by running:

```bash
sysctl net.ipv4.ip_forward
```

It should return `net.ipv4.ip_forward = 1`.

### Basic UFW Configuration

#### Configuring Default Policies

Start with a secure default policy:

```bash
# Set default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

#### Configure UFW to Accept Forwarded Packets

UFW by default blocks forwarded packets, which would prevent our port forwarding from working. Enable packet forwarding in UFW by editing its configuration:

```bash
sudo nano /etc/default/ufw
```

Find the `DEFAULT_FORWARD_POLICY` line and change it from `DROP` to `ACCEPT`:

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Save and close the file.

#### Opening Required Ports

Configure UFW to allow the necessary services:

```bash
# SSH access for administration
sudo ufw allow 22/tcp

# WireGuard VPN
sudo ufw allow 51820/udp

# HTTP/HTTPS for web interfaces and Let's Encrypt
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Mail server ports
sudo ufw allow 25/tcp  # SMTP
sudo ufw allow 587/tcp # Submission
sudo ufw allow 993/tcp # IMAPS
```

#### Restricting Access to Specific IPs

For enhanced security, certain services can be restricted to specific IP addresses. For example, allowing SMTP relay only from the Mailcow server via WireGuard:

```bash
# Allow SMTP from Mailcow server via WireGuard
sudo ufw allow from 10.0.0.2 to any port 2526/tcp
```

### Port Forwarding Configuration

#### Configuring NAT Rules in UFW

UFW uses iptables under the hood. To configure port forwarding, you need to edit the `/etc/ufw/before.rules` file, which contains rules that are processed before the UFW rules.

Add the following configuration to the top of the `/etc/ufw/before.rules` file, just after the `*nat` line:

```
*nat
:PREROUTING ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]

# Forward IMAPS (port 993) to Mailcow server via WireGuard
-A PREROUTING -p tcp --dport 993 -j DNAT --to-destination 10.0.0.2:993
-A POSTROUTING -p tcp -d 10.0.0.2 --dport 993 -j MASQUERADE

# Forward Submission (port 587) to Mailcow server via WireGuard
-A PREROUTING -p tcp --dport 587 -j DNAT --to-destination 10.0.0.2:587
-A POSTROUTING -p tcp -d 10.0.0.2 --dport 587 -j MASQUERADE

COMMIT

# Don't delete these required lines, otherwise there will be errors
*filter
```

Additionally, add the following forwarding rules to the `ufw-before-forward` chain in the same file:

```
# Forward traffic to Mailcow server
-A ufw-before-forward -p tcp -d 10.0.0.2 --dport 993 -j ACCEPT
-A ufw-before-forward -p tcp -d 10.0.0.2 --dport 587 -j ACCEPT
```

#### Explanation of Port Forwarding

The port forwarding configuration is critical for the email server architecture:

1. **Why Port Forwarding Is Needed**
   - Client email applications (Thunderbird, Outlook, etc.) need to authenticate with the mail server
   - These clients connect directly to ports 993 (IMAPS) and 587 (Submission)
   - Instead of implementing authentication on the proxy server, forwarding these connections to the Mailcow server is more efficient

2. **Benefits of This Approach**
   - Simplifies configuration and maintenance
   - Leverages Mailcow's robust authentication system
   - Ensures certificate validation works correctly
   - Uses the secure WireGuard tunnel for all sensitive communications

3. **Implementation Details**
   - `PREROUTING` rules redirect incoming traffic to the WireGuard interface
   - `POSTROUTING` rules with MASQUERADE handle the return traffic
   - Additional `ufw-before-forward` rules explicitly allow the forwarded connections

### Enabling and Verifying the Firewall

After configuring UFW, enable it and verify the configuration:

```bash
# Reload UFW to apply all changes
sudo ufw reload

# Enable UFW if not already enabled
sudo ufw enable

# Check UFW status and rules
sudo ufw status verbose
```

Expected output should show all configured rules with appropriate ports and services.

## Client Server Firewall Configuration

The client server requires a much simpler firewall configuration since it does not interact directly with the internet. It only needs to allow the WireGuard VPN connection.

### Basic UFW Configuration for Client Server

```bash
# Install UFW if not already installed
sudo apt update
sudo apt install ufw

# Set default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow WireGuard VPN traffic
sudo ufw allow 51820/udp

# Enable UFW
sudo ufw enable
```

That's it! The client server is now configured to:
1. Deny all incoming connections by default
2. Allow only WireGuard VPN traffic on UDP port 51820
3. Allow all outgoing connections

This minimal configuration ensures that the client server is completely invisible on the public internet, with the only access point being through the secure WireGuard tunnel from the proxy server.

## Security Considerations

1. **Proxy Server Security**
   - Only essential ports are opened to the internet
   - Legacy and insecure protocols are kept closed
   - Administrative access is restricted to SSH only
   - All forwarded connections go through the encrypted WireGuard tunnel

2. **Client Server Security**
   - Completely isolated from the internet
   - Only the WireGuard VPN port is open
   - All email services are only accessible through the encrypted tunnel
   - Authentication happens on the internal Mailcow server, not the exposed proxy

## Next Steps

**You are at Step 2 of 7 in the setup process**

After configuring your firewall:

➡️ If you haven't already, proceed to **Step 3: [Configure the WireGuard VPN](wireguard.md)**

If you've completed the previous steps:
- Verify all services are accessible through the firewall
- Test email client connectivity
- Ensure all ports are properly secured 