# Secure Mail Server Documentation 📧🔒

Welcome to the **Secure Mail Server Docs** repository! This project provides comprehensive documentation for setting up a private and secure mail server. Our guides cover everything you need to know about configuring your mail server using Haraka, Mailcow, DNS settings, firewall rules, Wireguard VPN, and Caddy for enhanced security and reliability.

[![Download Releases](https://img.shields.io/badge/Download%20Releases-blue.svg)](https://github.com/mariasaragih/secure-mail-server-docs/releases)

## Table of Contents

1. [Introduction](#introduction)
2. [Getting Started](#getting-started)
3. [Setting Up Haraka](#setting-up-haraka)
4. [Configuring Mailcow](#configuring-mailcow)
5. [DNS Configuration](#dns-configuration)
6. [Firewall Rules](#firewall-rules)
7. [Wireguard VPN Setup](#wireguard-vpn-setup)
8. [Caddy Configuration](#caddy-configuration)
9. [Security Best Practices](#security-best-practices)
10. [Contributing](#contributing)
11. [License](#license)

## Introduction

In today’s digital world, maintaining privacy and security in email communication is crucial. This documentation aims to guide you through the process of setting up a secure mail server that you can host yourself. With the right tools and configurations, you can ensure that your emails remain private and secure.

## Getting Started

To begin, you will need a server running a Linux distribution. This guide assumes you have basic knowledge of Linux commands. Ensure that your server meets the following requirements:

- A Linux server (Ubuntu, CentOS, etc.)
- Sufficient resources (CPU, RAM, Disk Space)
- Basic understanding of networking

### Prerequisites

1. **Domain Name**: You need a registered domain name for your mail server.
2. **Server Access**: SSH access to your server.
3. **Root Privileges**: Ensure you have root or sudo privileges on your server.

## Setting Up Haraka

Haraka is a lightweight, high-performance SMTP server written in Node.js. Follow these steps to set it up:

1. **Install Node.js**:
   ```bash
   sudo apt update
   sudo apt install nodejs npm
   ```

2. **Install Haraka**:
   ```bash
   sudo npm install -g Haraka
   ```

3. **Create a Haraka Instance**:
   ```bash
   mkdir /etc/haraka
   cd /etc/haraka
   haraka -i .
   ```

4. **Configure Haraka**:
   Edit the `config/smtp.ini` file to set up your SMTP settings.

5. **Start Haraka**:
   ```bash
   haraka -c /etc/haraka
   ```

For detailed configuration options, refer to the [Haraka documentation](https://haraka.github.io/).

## Configuring Mailcow

Mailcow is a full-featured mail server suite. Here’s how to set it up:

1. **Clone Mailcow Repository**:
   ```bash
   git clone https://github.com/mailcow/mailcow-dockerized.git
   cd mailcow-dockerized
   ```

2. **Configure Mailcow**:
   Copy the sample configuration file:
   ```bash
   cp mailcow.conf.example mailcow.conf
   ```

   Edit `mailcow.conf` to include your domain and other settings.

3. **Start Mailcow**:
   ```bash
   docker-compose up -d
   ```

4. **Access Mailcow Admin Panel**:
   Open your web browser and navigate to `http://your-domain.com`.

For more details, visit the [Mailcow documentation](https://mailcow.email/).

## DNS Configuration

Proper DNS configuration is vital for your mail server to function correctly. Here are the essential DNS records you need:

1. **A Record**: Points your domain to your server’s IP address.
2. **MX Record**: Directs email to your mail server.
3. **SPF Record**: Helps prevent spammers from sending messages on behalf of your domain.
4. **DKIM Record**: Provides a method for validating the authenticity of your emails.
5. **DMARC Record**: Helps protect your domain from unauthorized use.

Example DNS records:
```
A record: mail.your-domain.com -> your-server-ip
MX record: your-domain.com -> mail.your-domain.com
SPF record: v=spf1 a mx ~all
DKIM record: (your DKIM key)
DMARC record: v=DMARC1; p=none; rua=mailto:postmaster@your-domain.com
```

## Firewall Rules

Setting up firewall rules is crucial for securing your mail server. Here’s how to configure your firewall:

1. **Allow SMTP**:
   ```bash
   sudo ufw allow 25
   ```

2. **Allow IMAP**:
   ```bash
   sudo ufw allow 143
   ```

3. **Allow POP3**:
   ```bash
   sudo ufw allow 110
   ```

4. **Allow HTTPS (for Caddy)**:
   ```bash
   sudo ufw allow 443
   ```

5. **Enable the Firewall**:
   ```bash
   sudo ufw enable
   ```

## Wireguard VPN Setup

Using a VPN adds an extra layer of security. Follow these steps to set up Wireguard:

1. **Install Wireguard**:
   ```bash
   sudo apt install wireguard
   ```

2. **Generate Keys**:
   ```bash
   wg genkey | tee privatekey | wg pubkey > publickey
   ```

3. **Configure Wireguard**:
   Edit the configuration file `/etc/wireguard/wg0.conf`:
   ```
   [Interface]
   PrivateKey = <your-private-key>
   Address = 10.0.0.1/24

   [Peer]
   PublicKey = <peer-public-key>
   AllowedIPs = 10.0.0.2/32
   ```

4. **Start Wireguard**:
   ```bash
   sudo wg-quick up wg0
   ```

5. **Enable Wireguard on Boot**:
   ```bash
   sudo systemctl enable wg-quick@wg0
   ```

For more details, refer to the [Wireguard documentation](https://www.wireguard.com/).

## Caddy Configuration

Caddy is a powerful web server that automatically manages SSL certificates. Here’s how to set it up:

1. **Install Caddy**:
   ```bash
   sudo apt install caddy
   ```

2. **Create a Caddyfile**:
   Create a file at `/etc/caddy/Caddyfile` with the following content:
   ```
   your-domain.com {
       reverse_proxy localhost:your-mail-server-port
   }
   ```

3. **Start Caddy**:
   ```bash
   sudo systemctl start caddy
   ```

4. **Enable Caddy on Boot**:
   ```bash
   sudo systemctl enable caddy
   ```

For further information, visit the [Caddy documentation](https://caddyserver.com/docs/).

## Security Best Practices

1. **Regular Updates**: Keep your server and software updated.
2. **Use Strong Passwords**: Enforce strong password policies for user accounts.
3. **Backup Regularly**: Implement a backup strategy for your mail server data.
4. **Monitor Logs**: Regularly check server logs for unusual activity.
5. **Implement 2FA**: Use two-factor authentication where possible.

## Contributing

We welcome contributions to improve this documentation. If you find any issues or have suggestions, please open an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

For the latest releases, check the [Releases section](https://github.com/mariasaragih/secure-mail-server-docs/releases). 

[![Download Releases](https://img.shields.io/badge/Download%20Releases-blue.svg)](https://github.com/mariasaragih/secure-mail-server-docs/releases)

Thank you for using the Secure Mail Server Docs! We hope this documentation helps you set up a secure and reliable mail server.