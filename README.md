# pfsense-Multi-subnet-Lab
This is a virtual lab using VMware with pfsense based routing, firewalls rules and inter-subnet communications between isolated networks.

# pfsense Multi-Subnet Routing & Firewall Lab
This lab demonstrates how i **deployed pfsense as a router and firewall** to connect and control traffic between **two isolated** IPv4 subnets in a virtualized enviroment on VMware. 

## Lab Objectives
- Gain hands on experience on **configuring firewall rules** with multiple interfaces.
- Gain experience in making isolated networks.
- Enable **inter-subnet routing**
- Verify connectivity between isolated networks.

## Network Topology 
• **LAN Subnet**: 192.168.10.0/24
 - **pfsense LAN**: 192.168.10.1
 - **Ubuntu Server**: 192.168.10.3

• **OPT1 Subnet**: 192.168.20.0/24
 - **pfsense OPT1**: 192.168.20.0
 - **Kali Linux**: 192.168.20.3

## Enviroment
- VMware Workstation pro
- pfsense CE
- Kali Linux
- Unumtu Server

## Key Configurations
- pfsense configured with LAN and OPT! interfaces.
- Firewall rules permitting **inter-subnet traffic.**
- **Static IP** addressing with pfsense as default gateway. 
- VMware **custom vmnets** used as isolated networks.

## Verification & Testing
- Sucessful **ICMP ping** from kali to Ubuntu.
- Sucessful **ICMP ping** from Ubuntu to Kali.
- pfsense diagnostics confirming routing functionality.

## Skills Demonstrated
- Subnetting and interface segmentation.
- Inter-subnet routing and testing.
- Network troubleshooting and diagnostics.

## Issues encountered 
- Initial difficulty accessing the pfsense webgui due to VMware vmnets not being assined the correct ip address.
- Interface-subnet ICMP traffic blocked between OPT1(Kali) and LAN(Ubuntu) until firewall rules were corrected.
- Asymmetric connectivity observed(LAN OPT1 worked, OPT1 LAN failed).
- DNS resolution issues observed on host machine when pfsense acted as DNS without upstream resolvers configured.
