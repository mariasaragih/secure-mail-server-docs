# Firewall Configuration

This document describes the firewall configuration required for the secure email server architecture.

## Overview

A properly configured firewall is essential for securing your email server. This guide uses Uncomplicated Firewall (UFW) on the proxy server to:

1. Control which ports are accessible from the internet
2. Configure port forwarding for specific mail-related services
3. Establish secure rules for WireGuard VPN communication

## UFW Installation

UFW is usually pre-installed on most Ubuntu/Debian systems. If not, install it:

```bash
sudo apt update
sudo apt install ufw
```

## Enable IP Forwarding

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

## Basic UFW Configuration

### Configuring Default Policies

Start with a secure default policy:

```bash
# Set default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Configure UFW to Accept Forwarded Packets

UFW by default blocks forwarded packets, which would prevent our port forwarding from working. Enable packet forwarding in UFW by editing its configuration:

```bash
sudo nano /etc/default/ufw
```

Find the `DEFAULT_FORWARD_POLICY` line and change it from `DROP` to `ACCEPT`:

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Save and close the file.

### Opening Required Ports

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

### Restricting Access to Specific IPs

For enhanced security, certain services can be restricted to specific IP addresses. For example, allowing SMTP relay only from the Mailcow server via WireGuard:

```bash
# Allow SMTP from Mailcow server via WireGuard
sudo ufw allow from 10.0.0.2 to any port 2526/tcp
```

## Port Forwarding Configuration

### Configuring NAT Rules in UFW

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

### Explanation of Port Forwarding

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

## Enabling and Verifying the Firewall

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

## Security Considerations

1. **Minimal Port Exposure**
   - Only essential ports are opened to the internet
   - Legacy and insecure protocols are kept closed
   - Administrative access is restricted to SSH only

2. **Secure Forwarding**
   - All forwarded connections go through the encrypted WireGuard tunnel
   - Authentication happens on the internal Mailcow server, not the exposed proxy

## Next Steps

**You are at Step 2 of 7 in the setup process**

After configuring your firewall:

➡️ If you haven't already, proceed to **Step 3: [Configure the WireGuard VPN](wireguard.md)**

If you've completed the previous steps:
- Verify all services are accessible through the firewall
- Test email client connectivity
- Ensure all ports are properly secured 