# Haraka Mail Server Configuration

## Introduction

This document describes the setup and configuration of [Haraka](https://haraka.github.io/), a lightweight, high-performance SMTP server built on Node.js for the mail proxy server architecture.

### Why Haraka?

Simple port forwarding is insufficient for properly proxying email between a protected mail server and the internet. A dedicated Mail Transfer Agent (MTA) is required to handle the complexities of email routing while preserving essential sender information.

While traditional MTAs like Postfix are powerful, Haraka was chosen for this setup for several key reasons:

1. **XCLIENT Support**: Haraka has excellent support for the XCLIENT protocol extension, which is essential for preserving the original sender's IP address and information when forwarding emails. This ensures that emails appear to come from their original sources, not from the proxy server.

2. **Minimal Resource Footprint**: Haraka's event-driven architecture means it performs exceptionally well on limited hardware. This setup runs efficiently on a minimalist VPS with just 1GB RAM, 1 CPU, and 10GB SSD storage.

3. **Plugin-Based Architecture**: Haraka follows a barebone philosophy where you start with a minimal core and add only the functionality you need through plugins. This aligns perfectly with our goal of maintaining a minimal, secure proxy server.

4. **Modern JavaScript-Based Configuration**: Being built on Node.js, Haraka offers a more flexible and modern approach to mail server configuration compared to traditional MTAs.

## Overview

Haraka is configured to run two separate processes:
1. `haraka-in`: Handles incoming email and forwards it to Mailcow using XCLIENT
2. `haraka-out`: Handles outgoing email from Mailcow to external recipients

## Installation

### Tested Versions
The following versions have been tested and work well with this setup:
- Node.js version 22.14.0
- npm version 11.3.0

For Haraka specifically, version 3.0.0 is recommended as version 3.1.0 has some configuration issues where optional settings are being treated as required.

```bash
# Install Node.js and npm
apt update
apt install nodejs npm

# Install Haraka
npm install -g Haraka@3.0.0
```

## Configuration Structure

```
/opt/haraka/
├── config/
│   ├── haraka-in/
│   │   ├── config
│   │   └── plugins/
│   └── haraka-out/
│       ├── config
│       └── plugins/
```

## Haraka In Configuration

### Main Configuration (config/haraka-in/config)

```ini
[server]
listen=0.0.0.0:25
daemonize=true

[plugins]
xclient
queue/smtp_forward
rcpt_to.in_host_list
tls
```

### SMTP Configuration (config/haraka-in/smtp.ini)

```ini
; address to listen on (default: all IPv6 and IPv4 addresses, port 25)
; use "[::0]:25" to listen on IPv6 and IPv4 (not all OSes)
listen=[::]:25
```

Note: The `listen=[::]:25` setting configures Haraka to listen on port 25 for both IPv4 and IPv6 connections. This is actually the default setting, so the file can be left unmodified if you want to use the default behavior.

### Plugin Purposes

Each plugin in the configuration serves a specific role in the mail proxy architecture:

#### xclient
The [XCLIENT plugin](https://haraka.github.io/plugins/xclient) is essential for preserving the original sender information. When an email arrives at the proxy server, this plugin ensures that the original sender's IP address and details are forwarded to the backend mail server (Mailcow). Without this plugin, all emails would appear to originate from the proxy server's IP address, causing problems with spam filtering and authentication.

#### queue/smtp_forward
The [SMTP Forward plugin](https://haraka.github.io/plugins/queue/smtp_forward) handles the actual forwarding of emails to the backend mail server. It doesn't store messages locally but immediately forwards them to the specified destination (Mailcow). The queue prefix indicates that it operates at the queuing phase of mail processing.

#### rcpt_to.in_host_list
The [RCPT TO In Host List plugin](https://haraka.github.io/plugins/rcpt_to.in_host_list) validates incoming email recipient addresses against a list of accepted domains. It acts as a basic filter, ensuring that the proxy only processes emails destined for domains it's responsible for (specified in the host_list file). This prevents the server from accepting emails for domains it doesn't handle.

#### tls
The [TLS plugin](https://haraka.github.io/plugins/tls) enables encrypted connections, allowing the mail server to accept secure SMTP connections over TLS/SSL. This is essential for protecting email data in transit and is a requirement for modern email systems. It uses the certificate and private key specified in the configuration.

### SMTP Forward Plugin Configuration (config/haraka-in/plugins/smtp_forward.ini)

```ini
; Disable outbound to Internet
enable_outbound=false

; Forward everything to Mailcow via WireGuard
host=10.0.0.2
port=25
```

### XCLIENT Plugin Configuration (config/haraka-in/plugins/xclient.ini)

```ini
enabled=true
forward=true
destination=10.0.0.2:25
```

### TLS Plugin Configuration (config/haraka-in/plugins/tls.ini)

```ini
key=/path/to/private.key
cert=/path/to/certificate.crt
```

### SSL Certificates

This setup uses Let's Encrypt certificates with automatic renewal managed by acme.sh, configured to use IONOS API for DNS validation. This approach allows for certificate renewal without server downtime.

The same certificate files are used for both haraka-in and haraka-out instances, simplifying certificate management. Since both instances run on the same server, sharing certificates is a practical approach that reduces maintenance overhead.

#### Setting up acme.sh with IONOS API

1. Install acme.sh:
```bash
curl https://get.acme.sh | sh
```

2. Configure acme.sh to use IONOS API:
```bash
export IONOS_PREFIX="your-api-prefix"
export IONOS_SECRET="your-api-secret"

# Issue certificate using DNS validation with extended waiting time
~/.acme.sh/acme.sh --issue --dns dns_ionos -d mail.example.com --dnssleep 120
```

3. Create a combined PEM file (important for Haraka):
```bash
# Haraka's TLS plugin requires the private key and certificate to be in a single file
cat ~/.acme.sh/mail.example.com/mail.example.com.key \
    ~/.acme.sh/mail.example.com/mail.example.com.cer > /opt/haraka/config/haraka-in/tls/mail.example.com.pem
```

4. Set up automatic renewal with a deploy hook:
```bash
~/.acme.sh/acme.sh --deploy -d mail.example.com --deploy-hook haraka
```

5. Configure the path in tls.ini:
```ini
key=/opt/haraka/config/haraka-in/tls/mail.example.com.pem
cert=/opt/haraka/config/haraka-in/tls/mail.example.com.pem
```

Note: According to Haraka's TLS plugin documentation, the private key and certificate must be in the same file, which is why both key and cert parameters point to the same file.

### RCPT TO In Host List Plugin Configuration (config/haraka-in/plugins/rcpt_to.in_host_list.ini)

```ini
[main]
host_list=/opt/haraka/config/haraka-in/host_list
```

The host list file should contain the domain name used for email services. For example:
```
example.com
```

## Haraka Out Configuration

### Plugins Configuration (config/haraka-out/plugins)

```
relay
tls
```

### Host List Configuration (config/haraka-out/host_list)

```
mailrelay.example.com
```

### Relay Configuration (config/haraka-out/relay.ini)

```ini
[relay]
force_routing=true
acl=true
dest_domains=false
all=false
```

This configuration file controls how haraka-out handles relay decisions:

- `force_routing=true`: Forces messages to be routed according to configured routing rules rather than using standard MX lookups
- `acl=true`: Enables the Access Control List feature, which restricts relaying to only specific IP addresses
- `dest_domains=false`: Disables the destination domains feature, which would otherwise allow relaying to specific domains
- `all=false`: Ensures open relaying is disabled, which is critical for security

### Relay ACL Configuration (config/haraka-out/relay_acl_allow)

```
10.0.0.2/32
```

The `relay_acl_allow` file contains a list of IP addresses or networks (in CIDR notation) that are permitted to use haraka-out as a relay for sending mail to external destinations:

- `10.0.0.2/32`: This specifies the exact IP address of the Mailcow server on the WireGuard network
- The `/32` suffix indicates a single IP address rather than a network range
- Only mail coming from this specific IP will be accepted for relaying to external recipients
- Any connection from any other IP address attempting to send mail will be rejected

Without being listed in this file, no server (including the Mailcow server) would be able to send outbound mail through haraka-out.

### Understanding the Relay Plugin

The [relay plugin](https://haraka.github.io/plugins/relay) is a critical component for the outbound mail flow. It determines which mail can be relayed (forwarded) to external destinations. In this setup:

1. The plugin is configured with `acl=true` to use Access Control Lists, only allowing specific internal IPs (defined in `relay_acl_allow`) to send outbound mail.
2. `force_routing=true` enables mandatory routing rules.
3. `dest_domains=false` disables the destination domains feature.
4. `all=false` ensures that open relaying is disabled (security best practice).

### SMTP Configuration (config/haraka-out/smtp.ini)

```ini
listen=10.0.0.1:2526
```

#### Port Selection Strategy

Note that haraka-out uses port 2526 instead of the standard SMTP port 25. This is a deliberate design choice to prevent mail loopback:

1. If haraka-out were to listen on port 25, the same port as haraka-in, there's a risk that outbound messages could be mistakenly treated as inbound messages.
2. This could create a mail loop where messages cycle between haraka-in and haraka-out indefinitely.
3. Using port 2526 for outbound traffic ensures a clear separation between inbound and outbound mail flows.

The connection flow works as follows:
- Mailcow sends outbound mail to haraka-out on the WireGuard interface (10.0.0.1) port 2526
- haraka-out then relays this mail to the appropriate external recipients

### TLS Configuration (config/haraka-out/tls.ini)

```ini
key=/opt/haraka/config/haraka-out/tls/mailrelay.example.com.pem
cert=/opt/haraka/config/haraka-out/tls/mailrelay.example.com.pem
```

## Starting the Services

```bash
# Start Haraka In
haraka -c /opt/haraka/config/haraka-in

# Start Haraka Out
haraka -c /opt/haraka/config/haraka-out
```

## Systemd Service Files

### Haraka In Service (/etc/systemd/system/haraka-in.service)

```ini
[Unit]
Description=Haraka In Mail Server
After=network.target

[Service]
Type=simple
User=haraka
ExecStart=/usr/bin/haraka -c /opt/haraka/config/haraka-in
Restart=always

[Install]
WantedBy=multi-user.target
```

### Haraka Out Service (/etc/systemd/system/haraka-out.service)

```ini
[Unit]
Description=Haraka Out Mail Server
After=network.target

[Service]
Type=simple
User=haraka
ExecStart=/usr/bin/haraka -c /opt/haraka/config/haraka-out
Restart=always

[Install]
WantedBy=multi-user.target
```

## Security Best Practices

### User Permissions

It is recommended to create a dedicated `haraka` user in the `mail` group rather than running Haraka as root. This limits potential damage if the mail server is compromised.

```bash
# Create haraka user and add to mail group
sudo useradd -r -s /bin/false -m -d /opt/haraka -g mail haraka

# Set proper ownership of Haraka directories
sudo chown -R haraka:mail /opt/haraka
```

### Port 25 Binding

Port 25 is a privileged port (below 1024) and requires root privileges to bind. Since Haraka should not run as root, there are several ways to allow a non-root user to bind to port 25:

#### Option 1: Using setcap

```bash
# Allow the Node.js binary to bind to privileged ports
sudo setcap 'cap_net_bind_service=+ep' $(which node)
```

#### Option 2: Using AuthBind

AuthBind allows non-root users to bind to privileged ports:

```bash
# Install AuthBind
sudo apt-get install authbind

# Configure AuthBind for port 25
sudo touch /etc/authbind/byport/25
sudo chmod 500 /etc/authbind/byport/25
sudo chown haraka:mail /etc/authbind/byport/25
```

When using AuthBind, the Haraka service needs to be started differently:

```bash
# Manual start with AuthBind
authbind --deep /usr/bin/haraka -c /opt/haraka/config/haraka-in
```

For systemd services using AuthBind, modify the ExecStart line:

```ini
ExecStart=/usr/bin/authbind --deep /usr/bin/haraka -c /opt/haraka/config/haraka-in
```

> **Note**: When using AuthBind, make sure to update the ExecStart line in both service files (/etc/systemd/system/haraka-in.service and /etc/systemd/system/haraka-out.service) to include the authbind command as shown above.

### Service Registration

Create systemd service files to manage Haraka services:

### Haraka In Service (/etc/systemd/system/haraka-in.service)

```ini
[Unit]
Description=Haraka In Mail Server
After=network.target

[Service]
Type=simple
User=haraka
Group=mail
ExecStart=/usr/bin/haraka -c /opt/haraka/config/haraka-in
Restart=always
WorkingDirectory=/opt/haraka

[Install]
WantedBy=multi-user.target
```

### Haraka Out Service (/etc/systemd/system/haraka-out.service)

```ini
[Unit]
Description=Haraka Out Mail Server
After=network.target

[Service]
Type=simple
User=haraka
Group=mail
ExecStart=/usr/bin/haraka -c /opt/haraka/config/haraka-out
Restart=always
WorkingDirectory=/opt/haraka

[Install]
WantedBy=multi-user.target
```

### Enable and Start Services

```bash
# Enable services to start at boot
sudo systemctl enable haraka-in.service
sudo systemctl enable haraka-out.service

# Start services
sudo systemctl start haraka-in.service
sudo systemctl start haraka-out.service

# Check status
sudo systemctl status haraka-in.service
sudo systemctl status haraka-out.service
```

## Email Flow

1. **Incoming Email Flow**
   ```
   External Server -> Haraka In -> XCLIENT -> Mailcow
   ```

2. **Outgoing Email Flow**
   ```
   Mailcow -> WireGuard -> Haraka Out -> External Server
   ```

## Security Considerations

1. **Access Control**
   - Haraka In only accepts connections from the internet
   - Haraka Out only accepts connections from the WireGuard tunnel

2. **Header Preservation**
   - XCLIENT plugin preserves original sender information
   - SMTP Forward plugin maintains email headers


## Next Steps

**You are at Step 6 of 7 in the setup process**

Now that you have configured the Haraka mail server on your proxy server, proceed to:

➡️ **Step 7: [Set up Caddy web server](caddy.md)** on the proxy server

After completing all steps, make sure to:
- Test email flow in both directions
- Verify all services are running correctly
- Check your email server configuration with external tools like MXToolbox 