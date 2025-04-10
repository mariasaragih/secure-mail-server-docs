# Private and Secure Email Server Architecture

## The Journey of Self-Hosting Email

When I first embarked on the journey of self-hosting my email server, I encountered a common narrative in the tech community: "Don't do it." Friends and YouTube experts alike warned me about two major challenges:

1. **Complexity**: Setting up a private email server is notoriously complex, they said. And they weren't entirely wrong. The process is indeed complex, but perhaps not for the reasons we think. The complexity isn't inherent to the technology itself, but rather stems from a lack of modern, user-friendly tools and documentation. It's almost as if the tech industry has collectively decided that self-hosting email is a lost cause, leading to a self-fulfilling prophecy where the tools and guides remain outdated and complicated.

2. **Deliverability**: "Your emails will never reach inboxes," they cautioned. "Major providers like Gmail and Outlook maintain strict whitelists and blacklists, automatically flagging emails from residential IPs as spam." This, however, turned out to be a myth. Through careful architecture and proper configuration, it's not only possible but achievable to reach 100% deliverability rates - sometimes even better than commercial solutions. The key lies in understanding the email ecosystem and implementing the right infrastructure.

This project documents the current state of self-hosted email infrastructure, serving as a comprehensive reference for both immediate implementation and future development. By mapping out each component and requirement in detail, we create a foundation that can be used to develop more automated and simplified solutions. The architecture achieves excellent deliverability without relying on external email relay services, proving that complete email independence is possible. While the current implementation requires technical expertise, this guide aims to facilitate the creation of more user-friendly tools that will make self-hosted email accessible to everyone.

This repository contains documentation for a secure email server architecture implementation. The system consists of a client-server setup with secure communication through WireGuard VPN.

## Architecture Overview

The system is composed of two main components:

1. **Client Server (Internal)**
   - Runs Mailcow-Dockerized
   - Not exposed to the internet
   - Communicates with the proxy server through WireGuard VPN

2. **Proxy Server (External)**
   - Has a static public IP
   - Exposed to the internet
   - Runs Haraka mail server
   - Handles incoming and outgoing email traffic
   - Forwards emails to the client server using XCLIENT

## Setup Guide - Step by Step

Follow these steps in order to set up your secure email server:

1. **[Network Configuration](docs/network.md)** - Network architecture overview and requirements
2. **[Firewall Configuration](docs/firewall.md)** - UFW setup and port forwarding
3. **[WireGuard VPN Setup](docs/wireguard.md)** - Secure tunnel configuration between servers
4. **[Mailcow Configuration](docs/mailcow.md)** - Internal mail server setup
5. **[DNS Configuration](docs/dns.md)** - Email delivery DNS setup
6. **[Haraka Mail Server Setup](docs/haraka.md)** - Proxy mail server configuration
7. **[Caddy Web Server Setup](docs/caddy.md)** - Reverse proxy for web access

Each document contains detailed instructions for its specific component and will guide you to the next step.

## Security Features

- Complete network isolation of the mail server
- Secure communication through WireGuard VPN
- Preservation of original sender IP and headers
- Minimal attack surface with controlled port exposure

## Testing Environment

This architecture has been tested using Debian 12 as the operating system for both client and server components. The proxy server was deployed on a [IONOS VPS](https://www.ionos.com/servers/vps), chosen for the following reasons:

- IONOS provides minimal VPS solutions starting at €1/month with 1GB of RAM
- IONOS VPS instances come with a static public IP address with excellent reputation
- The static IP significantly improves email deliverability and reduces the risk of emails being marked as spam

While this setup relies on IONOS as an intermediary for the proxy server, it offers advantages over dedicated SMTP relay services (such as [MailChannels](https://www.mailchannels.com/)). Although services like MailChannels claim to respect privacy by not logging email content and purging metadata logs after 30 days, they still collect metadata about senders and recipients, and potentially have access to email content.

By self-hosting the SMTP relay through a VPS, we maintain greater control over our email infrastructure. In the future, any hosting provider that offers static IP addresses with good reputation could be used, making it possible to move to a completely independent server setup while maintaining good email deliverability.

## License

This project is licensed under the Creative Commons Zero v1.0 Universal (CC0) License - see the [LICENSE](LICENSE) file for details.

## Contributing

This guide was written after successfully completing this project. However, I haven't verified step-by-step from scratch that all the instructions in this guide lead to the desired outcome. There might be missing steps or room for improvement in terms of security measures and functionality.

I am absolutely open to improving these steps if:
- I missed some important security measures
- I overlooked critical functionality
- I forgot any important steps in the process
