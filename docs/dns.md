# DNS Configuration

This document describes the DNS configuration required for the secure email server architecture.

## DNS Records for Email

### MX Records (Mail Exchange)
MX records specify which mail servers are responsible for accepting email messages on behalf of your domain. They are essential for email delivery, as they tell other mail servers where to send emails addressed to your domain.

- **Purpose**: Direct email to the correct mail server
- **Format**: Priority number followed by the hostname of the mail server
- **Example**: `example.com. IN MX 10 mail.example.com.` (10 is the priority; lower numbers have higher priority)
- **Importance**: Without proper MX records, your domain cannot receive email

### A Records (Address)
A records map a domain name to an IPv4 address. For email servers, they provide the IP address where your mail server can be reached.

- **Purpose**: Define the IP address of your mail server
- **Format**: Domain name pointing to an IPv4 address
- **Example**: `mail.example.com. IN A 203.0.113.1`
- **Importance**: Essential for connecting domain names to the actual server IP

### SPF Records (Sender Policy Framework)
SPF records help prevent email spoofing by specifying which mail servers are authorized to send email on behalf of your domain.

- **Purpose**: Prevent email spoofing and reduce spam
- **Format**: TXT record with a specific syntax
- **Example**: `example.com. IN TXT "v=spf1 mx -all"`
  - `v=spf1`: SPF version
  - `mx`: Authorize all mail servers listed in the domain's MX records to send mail
  - `-all`: Strict policy (fail emails not matching the rule)
- **Importance**: Improves deliverability and helps prevent domain spoofing

The SPF record with `mx` parameter has several advantages:
- Authorization is automatically granted to servers specified in MX records
- No need to update the SPF record when the mail server's IP changes
- Simplifies DNS management because only the MX record needs modification

### DKIM Records (DomainKeys Identified Mail)
DKIM adds a digital signature to emails that recipients can verify using your public key published in DNS.

- **Purpose**: Email authentication to verify sender identity and message integrity
- **Format**: TXT record under a selector subdomain
- **Example**: `dkim._domainkey.example.com. IN TXT "v=DKIM1;k=rsa;t=s;s=email;p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA..."`
  - `v=DKIM1`: DKIM version
  - `k=rsa`: Key type (RSA in this case)
  - `t=s`: Testing flag (s indicates the key is not in testing mode)
  - `s=email`: Service type
  - `p=...`: The public key used to verify signatures
- **Importance**: Proves email authenticity and improves deliverability

### DMARC Records (Domain-based Message Authentication, Reporting, and Conformance)
DMARC builds on SPF and DKIM to provide clear instructions on how to handle emails that fail authentication.

- **Purpose**: Define policy for handling authentication failures and receive reports
- **Format**: TXT record with specific policy directives
- **Example**: `_dmarc.example.com. IN TXT "v=DMARC1; p=reject; rua=mailto:postmaster@example.com"`
  - `v=DMARC1`: DMARC version
  - `p=reject`: Policy (reject unauthenticated emails)
  - `rua=mailto:postmaster@example.com`: Address where aggregate reports should be sent
- **Importance**: Provides clear instructions to receiving servers and valuable feedback reports

### SRV Records (Service)
SRV records specify the location of specific services. For email, they're particularly useful for IMAP, SMTP, and other mail-related protocols.

- **Purpose**: Define connection points for specific services and help clients discover services automatically
- **Format**: `_service._protocol.example.com. IN SRV priority weight port target`
- **Examples**:
  ```
  _imap._tcp.example.com.       IN    SRV    10 0 993 mail.example.com.
  _imaps._tcp.example.com.      IN    SRV    10 0 993 mail.example.com.
  _submission._tcp.example.com. IN    SRV    10 0 587 mail.example.com.
  ```

Common mail-related SRV records:
- **_submission._tcp**: SMTP mail submission (port 587)
- **_imap._tcp**: IMAP service (typically port 143 or 993 for secure IMAP)
- **_imaps._tcp**: Secure IMAP service (port 993)

For example, in `_submission._tcp.example.com. IN SRV 10 0 587 mail.example.com.`:
- **_submission**: The service name (email submission)
- **_tcp**: The protocol used
- **10**: Priority (lower values have precedence)
- **0**: Weight for load balancing (when there are multiple servers with the same priority)
- **587**: Port number
- **mail.example.com**: Target hostname

### PTR Records (Pointer)
PTR records provide reverse DNS lookup, mapping an IP address back to a domain name.

- **Purpose**: Reverse DNS verification
- **Format**: Reversed IP address in the in-addr.arpa domain pointing to a hostname
- **Example**: `1.113.0.203.in-addr.arpa. IN PTR mail.example.com.`
- **Importance**: Many mail servers check for valid PTR records when receiving email

#### Setting Up PTR Records
Unlike other DNS records, PTR records are not configured in your domain's DNS settings. They must be set up at the server or hosting provider level:

1. **Server Provider Configuration**: 
   - PTR records are managed by the owner of the IP address (typically your hosting provider)
   - You need to access your server provider's control panel to configure them

2. **Example for Ionos VPS**:
   - Log in to your Ionos control panel
   - Navigate to "Network" → "Public IP"
   - Find the "Reverse DNS" option for your IP address
   - Enter the hostname that should be associated with this IP (e.g., mail.example.com)

3. **Important Considerations**:
   - The hostname in the PTR record should match the A record of your mail server
   - For example, if mail.example.com points to 203.0.113.1, then 203.0.113.1 should have a PTR record pointing to mail.example.com
   - Mismatch between A and PTR records can cause email delivery issues
   - Some mail servers may reject emails if the sending server doesn't have a matching PTR record

4. **Verification**:
   - After setting up the PTR record, you can verify it using:
   ```bash
   dig -x 203.0.113.1
   ```
   - Or use MXToolbox (https://mxtoolbox.com/ReverseLookup.aspx) to verify PTR records and other DNS settings

### CNAME Records (Canonical Name)
CNAME records create an alias from one domain name to another. For email, they can be used for service autodiscovery.

- **Purpose**: Create domain aliases
- **Format**: Alias domain pointing to a canonical domain name
- **Example**: `autoconfig.example.com. IN CNAME mail.example.com.`
- **Importance**: Useful for service discovery and standardization

### Autodiscovery Records
Autodiscovery records help email clients automatically configure themselves with the correct server settings.

```
autoconfig.example.com.  IN    CNAME    mail.example.com.
autodiscover.example.com. IN   CNAME    mail.example.com.
```

These records support:
- **autoconfig**: Mozilla Thunderbird's autodiscovery protocol
- **autodiscover**: Microsoft Outlook/Exchange autodiscovery protocol

## DKIM Configuration in MailCow

To configure DKIM in MailCow:

1. First, you need to configure the domain in MailCow
2. After configuring the domain, access the "Domains" section and click the "Edit" button (pencil icon) next to the domain
3. In the domain edit panel, scroll to the bottom of the page
4. The complete DKIM key will appear in the last section of the domain edit page (in the section labeled "Domain: example.com (dkim_domainkey)")
5. The displayed value is the DKIM public key already formatted correctly to be inserted as a TXT record in DNS
6. Copy this key and insert it into the DNS TXT record for the configured DKIM selector

## Testing DNS Configuration

### Using MXToolbox
MXToolbox is the recommended tool for comprehensive email DNS testing:

1. **MX Record Check**: 
   - Visit https://mxtoolbox.com/SuperTool.aspx
   - Enter "mx:example.com" to check MX records

2. **SPF Testing**:
   - Visit https://mxtoolbox.com/SPFRecordGenerator.aspx
   - Enter your domain to validate SPF records and get suggestions

3. **DKIM Testing**:
   - Visit https://mxtoolbox.com/dkim.aspx
   - Enter selector._domainkey.example.com to verify DKIM setup

4. **DMARC Testing**:
   - Visit https://mxtoolbox.com/DMARC.aspx
   - Enter _dmarc.example.com to verify DMARC policy

5. **Blacklist Check**:
   - Visit https://mxtoolbox.com/blacklists.aspx
   - Enter your mail server IP to check against common blacklists

6. **PTR Lookup**:
   - Visit https://mxtoolbox.com/ReverseLookup.aspx
   - Enter your mail server IP address to verify the PTR record

All tests should return positive results before proceeding with mail server configuration.

### Using Mail Tester
Mail Tester is an excellent tool to check the overall deliverability score of your email setup:

1. **Visit Mail Tester**:
   - Go to https://www.mail-tester.com/
   - The site will generate a unique email address for testing

2. **Send a Test Email**:
   - Send an email from your mail server to the generated address
   - Use the same settings and headers that your regular emails would use
   - Include typical content (signature, links, images) to test thoroughly

3. **Review Your Score**:
   - After sending the email, refresh the Mail Tester page
   - You'll receive a score out of 10, with detailed feedback on:
     - DNS configuration (SPF, DKIM, DMARC)
     - Server configuration
     - Content quality
     - Presence on blacklists
     - Authentication

4. **Interpreting Results**:
   - A score of 9-10: Excellent deliverability
   - A score of 7-8: Good, but some improvements needed
   - A score below 7: Significant issues to address
   - Each issue will have specific recommendations for fixing

### Checking DNS Propagation
DNS changes can take time to propagate worldwide. You can verify propagation status using these tools:

1. **DNS Checker**:
   - Visit https://dnschecker.org/
   - Enter your domain name and select the record type (A, MX, TXT, etc.)
   - Check propagation across multiple global DNS servers

2. **What's My DNS**:
   - Visit https://whatsmydns.net/
   - Enter your domain and select the record type
   - View a global map showing propagation status and how your DNS records appear from different locations worldwide

3. **Propagation Times**:
   - TTL (Time to Live) settings affect propagation speed
   - Standard propagation can take 24-48 hours
   - Some changes may propagate faster (4-8 hours)
   - Always allow sufficient time before troubleshooting

### Command-line Testing

```bash
# SPF Testing
dig TXT example.com

# DKIM Testing
dig TXT dkim._domainkey.example.com

# DMARC Testing
dig TXT _dmarc.example.com
```

## Next Steps

**You are at Step 5 of 7 in the setup process**

Now that you have configured DNS records for your domain, proceed to:

➡️ **Step 6: [Set up Haraka mail server](haraka.md)** on the proxy server

Future steps:
7. Configure [Caddy web server](caddy.md)