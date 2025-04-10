# Network Configuration

This document describes the network setup required for the secure email server architecture.

## Network Architecture

```
Internet <---> Proxy Server (Public IP) <---> WireGuard VPN <---> Client Server (Internal)
```

## Port Requirements

### Proxy Server (External)
- Port 25 (TCP): Incoming SMTP traffic
- Port 587 (TCP): Submission (STARTTLS)
- Port 993 (TCP): IMAPS
- Port 51820 (UDP): WireGuard VPN
- Port 22 (TCP): SSH access
- Port 80 (TCP): HTTP (Let's Encrypt challenge)
- Port 443 (TCP): HTTPS (Autoconfig via Caddy)

Note: Following a minimal exposure approach, legacy protocols like POP3, POP3S, unencrypted IMAP, and SMTPS have been disabled to reduce the attack surface.

### Client Server (Internal)
- **Only Port 51820 (UDP)**: WireGuard VPN
- No other ports exposed to the internet
- The client server is completely invisible on the public network except for the WireGuard VPN port

## Network Security Considerations

1. **Firewall Configuration**
   - **Proxy Server**: Should only expose necessary mail-related ports as listed above
   - **Client Server**: Should only allow WireGuard VPN traffic (UDP port 51820)
   - All other ports should be blocked on both servers

2. **IP Configuration**
   - Proxy server requires a static public IP
   - Client server can use private IP addressing
   - WireGuard VPN should use private IP addressing (e.g., 10.0.0.0/24)

3. **DNS Requirements**
   - MX records pointing to proxy server
   - SPF records for email authentication
   - DKIM configuration
   - DMARC policy

## Network Diagram

```
                    Internet
                       |
                       |
                [Proxy Firewall]
                       |
                Proxy Server
                (Public IP)
                       |
                [WireGuard VPN]
                       |
               [Client Firewall]
                       |
                Client Server
                (Internal)
```

## Network Security Benefits

This architecture provides several key security benefits:

1. **Minimized Attack Surface**: 
   - The client server hosting Mailcow is completely invisible on the public internet
   - Only the WireGuard VPN port (UDP 51820) needs to be accessible
   - All other services are protected behind the proxy

2. **Defense in Depth**:
   - Even if the proxy server is compromised, the attacker would still need to breach the WireGuard VPN to access the mail data
   - Two separate firewalls (proxy and client) must be bypassed

3. **Simplified Management**:
   - The client server can be located anywhere (even behind NAT)
   - No need to expose multiple mail services directly to the internet

## Next Steps

**You are at Step 1 of 7 in the setup process**

Now that you understand the network architecture, proceed to:

➡️ **Step 2: [Configure the Firewall](firewall.md)**

Future steps:
3. Configure the [WireGuard VPN](wireguard.md)
4. Set up [Mailcow](mailcow.md) on the client server
5. Configure [DNS records](dns.md)
6. Set up [Haraka mail server](haraka.md)
7. Configure [Caddy web server](caddy.md) 