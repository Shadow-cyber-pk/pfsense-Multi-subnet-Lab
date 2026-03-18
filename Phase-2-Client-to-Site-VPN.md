# Deployment of Client-to-Site VPN: OpenVPN on pfSense and CGNAT Workaround

**Lab Date**: 28/02/2026  
**Skills Demonstrated**: pfSense, OpenVPN, VPN troubleshooting, Network diagnostics, Tailscale, CGNAT analysis

---

## Overview (TL;DR)

This lab documents the complete deployment of a client-to-site VPN on pfSense, 
root-cause analysis of a connection failure caused by ISP **CGNAT** blocking, and final 
resolution using Tailscale as a functional overlay solution. Key learning: OpenVPN 
configuration was correct; failure was environmental, not configuration-related.

---

## Network Architecture Overview

### Primary Internet Access Path

```
┌─────────────┐      ┌──────────┐      ┌───────────────┐      ┌─────────────┐
│   Client    │────▶│  CGNAT   │─────▶│ pfSense WAN   │────▶│ LAN Subnets │
│ (External)  │      │ (ISP)    │      │  (vmnet8)     │      │ (vmnet2)    │
└─────────────┘      └──────────┘      └───────────────┘      └─────────────┘
                           |
                           ▼
                  ✖️ Inbound blocked
```

**Data Flow**: External client → ISP CGNAT → pfSense WAN interface → Lab subnets  
**Security Model**: Stateful firewall with default-deny inbound policy

---

### Tailscale VPN Overlay Path

```
┌─────────────┐    ┌──────────────────────────┐    ┌────────────────────┐
│   Client    │    │  Tailscale Mesh Network  │    │  Ubuntu Router     │
│  (Remote)   │───▶│    (Encrypted Overlay)   │──▶│  (VPN Gateway)     │
└─────────────┘    └──────────────────────────┘    └────────────────────┘
                                                            │
                                                            ▼
                                                    ┌──────────────────┐
                                                    │  Lab Subnets     │
                                                    │ (vmnet2)         │
                                                    └──────────────────┘
```

**Data Flow**: Remote client → Encrypted mesh tunnel → Ubuntu router → Lab subnets  
**Security Model**: End-to-end encryption, mutual authentication

---


---

# Step 1: Create an Internal Certificate Authority (CA)

1. log in to pfSense WebGUI

2. Navigate to: System → Certificates → Add


## Configuration

- Descriptive Name: client-to-site-vpn-ca

- Method: Create an Internal Certificate Authority

- Randomize Serial: Enabled

- Key Type: ECDSA

- Curve: prime256v1

- Digest Algorithm: SHA512

- Lifetime: 1194 days

- Common Name: internal-ca

- Organization: myvirtualnetworkinglab

- Organizational Unit: pfsenselab


## Issues Encountered

- Error: Unable to create CA (unknown error)


## Resolution

- Identified instability when using certain configurations

- Successfully created CA using ECDSA

- Creation succeeded in a private browser session after repeated failures


## Certificate Revocation List (CRL)

- Name: vpn-crl

- Lifetime: 15 days

- Serial: 0



---

# Step 2: Create Server and Client Certificates

- Server Certificate

- Method: Internal Certificate

- Name: openvpn-server-cert

- CA: client-to-site-vpn-ca

- Key Type: ECDSA

- Curve: secp384r1

- Digest: SHA512

- Lifetime: 300 days


## Client Certificate

- Method: Internal Certificate
 
- Name: openvpn-client-cert

- CA: client-to-site-vpn-ca

- Key Type: ECDSA

- Curve: secp384r1

- Digest: SHA512

- Lifetime: 1194 days



---

# Step 3: Configure OpenVPN Server

Navigate to: VPN → OpenVPN → Servers → Add

## General

- Description: client-to-site-vpn


## Endpoint

- Protocol: UDP (IPv4 only)

- Interface: WAN

- Port: 1194


# Cryptography

- TLS Mode: Encryption and Authentication

- ECDH Curve: secp384r1

- Data Encryption Algorithms: AES-256-GCM (primary)

- Auth Digest Algorithm: SHA512


(Note: DH parameters omitted. Not required for ECDSA.)

# Tunnel

- Tunnel Network: 192.168.30.0/24

- Local Networks: 192.168.10.0/24, 192.168.20.0/24

- Concurrent Connections: 1

- Compression: Disabled

- Inter-client Communication: Disabled

- Duplicate Connections: Disabled


## Client Settings

- Dynamic IP: Enabled

- Topology: Subnet


## Advanced

- Username as Common Name: Enabled

- Gateway Creation: IPv4

- Verbosity: 3



---

# Step 4: Firewall Rule (WAN)

Navigate to: Firewall → Rules → WAN

- Action: Pass

- Protocol: UDP

- Source: Any

- Destination: WAN Address

- Port: 1194

- Logging: Enabled


(Note: No NAT rule required for OpenVPN on pfSense.)


---

# Step 5: OpenVPN Client Configuration

Navigate to: VPN → OpenVPN → Clients → Add

## General

- Description: windows-client

- Server Mode: Peer-to-Peer (SSL/TLS)

- Device Mode: TUN


## Endpoint

- Protocol: UDP (IPv4 only)


## Authentication

- Username / Password configured


## Cryptography

- TLS Enabled

- TLS Key: Auto-generated

- Peer CA: client-to-site-vpn-ca

- CRL: vpn-crl

- Client Certificate: openvpn-client-cert

- Encryption: AES-256-GCM

- Auth Digest: SHA512


## Tunnel

- Compression: Disabled

- Topology: Subnet


## Advanced

- Exit Notify: Retry 1x

- Gateway Creation: IPv4

- Verbosity: 3



---

# Step 6: Client Export

1. Install package: System → Package Manager → openvpn-client-export


2. Navigate to: VPN → OpenVPN → Client Export



## Issue

- No download option available


## Root Cause

- Client certificate not linked to user


## Resolution

- Navigate to System → User Manager

- Edit user

- Assign or create certificate under user


Download and transfer the client configuration.


---

# Step 7: Connection Failure

## Error

TLS key negotiation failed to occur within 60 seconds


---

# Diagnostic Logic Chain

- TLS timeout → No response from server

- Firewall rule verified → Open

- Port forwarding configured → No effect

- NAT rule attempted → Incorrect approach

- Conclusion → Upstream network blocking traffic



---

# Root Cause: CGNAT

- ISP assigns shared public IPv4 (Carrier-Grade NAT)

- No dedicated public IP

- No inbound port ownership

- Port forwarding on local router ineffective


Result: OpenVPN server unreachable from internet


---

# Solution: Tailscale

## Setup

Installed on:

- Windows client

- Kali Linux

- Ubuntu server


All nodes joined same network.

## Routing

Configured Ubuntu server as subnet router between:

- Tailscale network

- Local lab subnets


## Verification

- Ping to lab IP before Tailscale: Failed

- Ping to Tailscale IP: Successful

- Ping to lab IP after routing: Successful



---

# Issues Encountered (Tailscale)

## Authentication Failure

### Cause

- Incorrect auth link

- Incorrect command syntax


### Resolution

- Accessed Ubuntu via SSH

- Ran:


sudo tailscale up

- Completed browser authentication


## SSH Issue

### Cause

- SSH not installed or enabled


### Resolution

sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh


---

# Final Outcome

- OpenVPN correctly configured but unusable under CGNAT

- Tailscale provided functional overlay solution

- Full connectivity achieved across all lab systems



---

# Key Takeaways

- CGNAT blocks inbound VPN connections

- OpenVPN requires public IP or port forwarding capability

- ECDSA eliminates need for DH parameters

- Consistent crypto settings are critical

- Certificate-to-user linking is required for client export

- Tailscale bypasses CGNAT via encrypted overlay networking

- Root-cause analysis is essential before reconfiguration

---

## Screenshots & Evidence

### pfSense Certificate Authority
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/censored-certificate-authority.jpeg)

### OpenVPN Server Certificate
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/censored-server-certificate.jpeg)

### windows openVPN client certificate
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/censored-client-certificate.jpeg)

### Tailscale Network Topology
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/censored-tailscale-admin-panel.jpeg)

### Ping Test Results
- Before tailscale was setup
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/censored-ping-fail.jpeg)

- After tailscale was setup
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/censored-ping-sucessful.jpeg)

### ping using tailscale ip
![image alt](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/ffe2a9cf2ba8dbcb8b920f9494d444ba13f14503/images/tailscale-ping.jpeg)
