# Mailcow Configuration

This document describes the setup and configuration of Mailcow-Dockerized on the client server.

## Overview

[Mailcow](https://mailcow.email/) is a comprehensive email server solution that runs in Docker containers. It represents the core set of services managing email on a client server not directly exposed to the internet, where emails are stored and managed. Mailcow includes:
- Postfix (SMTP)
- Dovecot (IMAP/POP3)
- Rspamd (Spam filtering)
- SOGo (Groupware/Integrated email client)
- Administration interface for managing domains, mailboxes, aliases, etc.

## Prerequisites

According to the [official documentation](https://docs.mailcow.email/):
- Docker Engine and Docker Compose
- Git
- 1 GHz CPU (minimum)
- 6 GB RAM (minimum) + 1 GB swap
- 20 GB disk space (excluding mail storage)
- x86_64 or ARM64 architecture
- A valid domain name pointing to your server

## Installation

The installation process is relatively simple, especially using Docker. First, ensure Docker Engine is installed on your system before proceeding with the following steps:

### Installing Docker Engine

Before installing Mailcow, you need to install Docker Engine. Refer to the [official Docker Engine installation documentation](https://docs.docker.com/engine/install/) for platform-specific instructions.

After installing Docker Engine, proceed with the Mailcow installation:

```bash
# Clone the repository
cd /opt
git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized

# Generate configuration
./generate_config.sh
```

For detailed and up-to-date installation instructions, always refer to the [official Mailcow documentation](https://docs.mailcow.email/).

## Starting Mailcow

```bash
# Start all containers
docker compose up -d

# Check status
docker compose ps
```

## Initial Setup

1. Access the admin interface:
   ```
   https://mail.example.com/admin
   ```

   Access the user interface:
   ```
   https://mail.example.com
   ```
   
   Note: In the latest version (2025-03), administrator credentials can only be used at the /admin endpoint, not at the root domain.

2. Login with default admin credentials:
   - Username: admin
   - Password: moohoo

   It is strongly recommended to change the default password immediately after the first login.

3. Add domain:
   - Go to "Mailbox" → "Add Domain"
   - Enter domain name
   - Configure DKIM

4. Create mailboxes:
   - Go to "Mailbox" → "Add Mailbox"
   - Create user accounts

## Updating Mailcow

To update your Mailcow installation, use the update script provided in the mailcow-dockerized directory:

```bash
# Navigate to mailcow directory
cd /opt/mailcow-dockerized

# Run the update script
./update.sh
```

The update script will:
1. Check for available updates
2. Show what changes will be applied
3. Handle merge conflicts automatically when possible
4. Restart the services after updating


For more detailed information, refer to the [official update documentation](https://docs.mailcow.email/maintenance/update/). 

## Next Steps

**You are at Step 4 of 7 in the setup process**

Now that you have configured Mailcow on the client server, proceed to:

➡️ **Step 5: [Configure DNS Records](dns.md)**

Future steps:
6. Set up [Haraka mail server](haraka.md)
7. Configure [Caddy web server](caddy.md)

After completing all steps, make sure to test the entire email flow and verify all services are running correctly.