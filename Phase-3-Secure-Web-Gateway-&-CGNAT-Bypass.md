# Secure Web Gateway & CGNAT Bypass
**Author**: Ali Hassan
**Date**: March 6, 2026
**Focus**: Networking, Infrastructure Hardening, Zero-Trust Tunneling
# 1. Executive Summary
This lab demonstrates the deployment of a secure, public-facing web server from within a private network restricted by Carrier-Grade NAT (CGNAT). By utilizing 
NGINX as a Reverse Proxy and Tailscale Funnel as a secure ingress point, I established a verified communication path from the public internet to a hardened 
local environment.
# 2. Environment Configuration
## Virtual Infrastructure
- **Security Gateway**: pfSense (WAN/LAN segmentation)
- **Web Host**: Ubuntu Server 24.04 (IP: 192.168.20.100)
- **Audit Node** : Kali Linux (Internal testing)
- **External Client**: Cellular Device (Public testing)
## Software Stack
- **Web Server**: Nginx 1.24.0
- **Firewall**: UFW (Uncomplicated Firewall)
- **Tunneling**: Tailscale (WireGuard-based)
# 3. Implementation Workflow
## Step 1: Web Engine & File System
- Established a dedicated web root: /var/www/virtual-lab-website/.
- Created a custom index.html to verify successful document delivery.
- Set directory permissions to allow the www-data user read-only access.
## Step 2: SSL/TLS & Cryptography
- Generated a **Self-Signed Certificate** via OpenSSL:
  - Key: 2048-bit RSA
  - Purpose: Local encryption for internal LAN testing (Port 443).
- Configured HSTS (301 Redirects): Forced all insecure Port 80 requests hitting the local IP to upgrade to HTTPS.
## Step 3: NGINX Hardening
- **Server Tokens**: Modified /etc/nginx/nginx.conf to set server_tokens off;.
  - Result: Nmap scans no longer disclose the NGINX version number.
- **Access Control**: Restricted server_name to specific IP/Hostnames to prevent Host Header injection attacks.
## Step 4: Bypassing CGNAT via Tailscale
- **Challenge**: ISP restrictions prevented inbound port forwarding on the physical router.
- **Solution**: Deployed **Tailscale Funnel**.
- **Architecture**:
Tailscale acts as a **Reverse Proxy** at the network edge.
Implemented **SSL Offloading**: Public TLS is terminated by Tailscale, then proxied to the local NGINX instance over an encrypted tunnel.
# 4. Troubleshooting & Logic Fixes
During the lab, several "real-world" networking hurdles were encountered and resolved:
- **The 404 Loop**: Discovered that Tailscale Funnel hits NGINX on Port 80 using the Tailscale hostname.
   - Fix: Updated the Port 80 server block to recognize the Tailscale FQDN and serve files instead of returning a 404.
- **UFW Permissions**: Whitelisted only the **Windows Tailscale IP** for SSH (Port 22) to adhere to the **Principle of Least Privilege**.
- **NTP Sync**: Corrected system time synchronization issues by setting the timezone to Asia/Karachi and forcing an NTP update to ensure accurate logging.
# 5. Verification (The Evidence)
## Nmap Reconnaissance Results
A scan from Kali against the public URL revealed a hardened posture:
- **Port 80/443**: Open (NGINX version hidden).
- **Port 22**: Open (Restricted to authorized IP only).
- **Service Detection**: Identifies as nginx (Generic), preventing targeted version-specific exploits.
- **Log Audit**
Verified successful traffic flow in /var/log/nginx/access.log:
- **HTTP 200**: Successful file delivery.
- **HTTP 301**: Successful secure redirect.
- **HTTP 404/403**: Verified firewall/proxy blocking unauthorized directory listing.
# 6. Conclusion
This project confirms that secure service delivery is possible even in restricted networking environments. It bridges the gap between **Classic Networking**
(Firewalls/Routing) and **Modern Cloud-Native Security** (Zero-Trust/Tunneling).

### Nmap scan after hardening
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/fcfa440f10eaeca3ea847f8deacea910d2963de4/images/Screenshot%202026-03-06%20040400.png)
