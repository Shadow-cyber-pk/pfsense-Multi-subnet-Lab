# Enterprise Multi-Subnet Network with pfSense Firewall & VPN

## Overview
Designed and implemented a simulated enterprise network using pfSense, featuring multiple subnets, firewall policies, and secure remote access via VPN.

## Network Topology
![Network Diagram](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/3e41798101ba6705cd2acd29f3ebfbfb3f6f970d/images/pfSense_Network_Topology.drawio.svg)


## Key Features
- Multi-subnet network segmentation
- Firewall rule enforcement (allow/deny traffic)
- Client-to-site VPN setup (OpenVPN)
- Secure web gateway configuration (NGINX)
- NAT & CGNAT bypass

## Tech Stack
- pfSense
- VMware Workstation
- Ubuntu Server
- Kali Linux
- Networking (TCP/IP, NAT, VPN)

## Key Achievements
- Successfully enabled inter-subnet routing with controlled firewall policies  
- Configured secure remote access via VPN  
- Implemented traffic filtering and segmentation between networks  
- Deployed and secured a web server behind firewall rules  

## Troubleshooting & Root Cause Analysis
- Resolved ICMP blocking between subnets by correcting firewall rules  
- Diagnosed asymmetric routing issues between LAN and OPT1 networks  
- Fixed DNS resolution failures caused by missing upstream resolver configuration  
- Identified VMware network misconfiguration affecting pfSense web interface access  

## Project Phases
* **[Phase 1: Network segmentation and access control](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/main/Phase-1-Controlled-internet-access.md)**
* **[Phase 2: VPN configuration and firewall hardening](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/main/Phase-1-Controlled-internet-access.md)**
* **[Phase 3: Secure web gateway deployment and NAT configuration](https://github.com/Shadow-cyber-pk/pfsense-Multi-subnet-Lab/blob/main/Phase-3-Secure-Web-Gateway-%26-CGNAT-Bypass.md)**
