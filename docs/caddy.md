# Caddy Web Server Configuration

This document describes the setup and configuration of Caddy web server for secure web access to the mail server administration interface.

## Overview

[Caddy](https://caddyserver.com/) is a modern, open-source web server with automatic HTTPS capabilities. In this setup, Caddy serves as a reverse proxy on the external proxy server, allowing secure access to the Mailcow administration interface which is running on the internal client server.

## Why Caddy?

Caddy offers several advantages over traditional reverse proxies like Nginx:

1. **Automatic HTTPS**: Caddy automatically obtains and renews SSL certificates from Let's Encrypt without any configuration.

2. **Simple Configuration**: Caddy uses a straightforward, human-readable configuration syntax that is easier to understand and maintain.

3. **Modern Security Defaults**: Caddy implements modern security practices by default, such as HTTP/2, TLS 1.3, and strong cipher suites.

4. **Zero Downtime Config Reloads**: Caddy can reload its configuration without dropping connections.

## Installation

Install Caddy on the proxy server:

```bash
# For Debian/Ubuntu
sudo apt update
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy

# For CentOS/RHEL
sudo dnf install 'dnf-command(copr)'
sudo dnf copr enable @caddy/caddy
sudo dnf install caddy
```

## Configuration

Create a Caddyfile at `/etc/caddy/Caddyfile` with the following configuration:

```
mail.example.com {
    encode gzip

    # Automatic SSL/TLS (Caddy handles certificates automatically)
    tls {
        protocols tls1.2 tls1.3
    }

    # Reverse proxy to Mailcow web interface
    reverse_proxy 10.0.0.2:8443 {
        transport http {
            tls_insecure_skip_verify
        }
    }

    # Security headers
    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        X-Frame-Options "SAMEORIGIN"
        Referrer-Policy "strict-origin-when-cross-origin"
    }
}

autoconfig.example.com {
    reverse_proxy 10.0.0.2:8443 {
        transport http {
            tls_insecure_skip_verify
        }
    }

    tls {
        protocols tls1.2 tls1.3
    }

    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        Referrer-Policy "no-referrer"
    }

    encode zstd gzip
}

autodiscover.example.com {
    reverse_proxy 10.0.0.2:8443 {
        transport http {
            tls_insecure_skip_verify
        }
    }

    tls {
        protocols tls1.2 tls1.3
    }

    header {
        Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
        Referrer-Policy "no-referrer"
    }

    encode zstd gzip
}
```

### Configuration Explanation

1. **Domain Blocks**: Each domain (`mail.example.com`, `autoconfig.example.com`, `autodiscover.example.com`) has its own configuration block.

2. **TLS Configuration**: 
   - `tls { protocols tls1.2 tls1.3 }` enforces modern TLS protocols.
   - Caddy automatically obtains and manages SSL certificates.

3. **Reverse Proxy**:
   - `reverse_proxy 10.0.0.2:8443` forwards requests to the Mailcow web interface on the internal server.
   - `tls_insecure_skip_verify` is necessary because Mailcow uses a self-signed certificate internally.

4. **Security Headers**:
   - `Strict-Transport-Security` enforces HTTPS connections.
   - `X-Frame-Options "SAMEORIGIN"` prevents clickjacking attacks by controlling which websites can embed your pages in frames. The "SAMEORIGIN" value ensures that your pages can only be embedded in frames on the same domain, preventing malicious sites from embedding your content to trick users into clicking on them.
   - `Referrer-Policy "strict-origin-when-cross-origin"` controls how much referrer information is shared when users navigate from your site to others. This particular setting sends the full URL when navigating within the same site, but only sends the origin (domain) when navigating to other sites, enhancing privacy by limiting information leakage while still providing essential analytics data.

5. **Compression**:
   - `encode gzip` or `encode zstd gzip` enables compression to reduce bandwidth usage.

## Network Considerations

Ensure that:
- Port 80 (HTTP) and 443 (HTTPS) are open on the proxy server firewall.
- The internal Mailcow server is accessible from the proxy server via the WireGuard VPN.
- DNS records for `mail.example.com`, `autoconfig.example.com`, and `autodiscover.example.com` all point to the proxy server's IP address.

## Starting and Managing Caddy

```bash
# Start Caddy service
sudo systemctl start caddy

# Enable Caddy to start at boot
sudo systemctl enable caddy

# Check status
sudo systemctl status caddy

# Reload configuration without restarting
sudo systemctl reload caddy
```

## Testing the Configuration

1. Visit `https://mail.example.com` in a web browser.
2. You should be redirected to the Mailcow login page.
3. Verify that the connection is secure (check for the padlock in your browser).
4. Test email client autoconfiguration with the appropriate email clients.

## Next Steps

**You are at Step 7 of 7 in the setup process**

Congratulations! You have completed the setup of your secure email server architecture.

After completing all steps:

1. **Test email flow** by sending and receiving test emails
2. **Verify security** by checking SPF, DKIM, and DMARC with [MXToolbox](https://mxtoolbox.com/)
5. **Test the web interface** by accessing mail.example.com through this Caddy reverse proxy 