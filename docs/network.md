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
- No ports exposed to the internet
- WireGuard VPN interface only
- Internal network access to Mailcow services

## Network Security Considerations

1. **Firewall Configuration**
   - Proxy server should only expose necessary mail-related ports
   - All other ports should be blocked
   - Internal client server should block all incoming traffic except WireGuard

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
                [Firewall Rules]
                       |
                Proxy Server
                (Public IP)
                       |
                [WireGuard VPN]
                       |
                Client Server
                (Internal)
```

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