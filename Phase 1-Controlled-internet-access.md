# Network Documentation

This document records the transition from an isolated multi-subnet lab environment to a routed, NAT-enabled network with controlled internet access via pfSense.

### **Network Topology**

- vmnet0: NAT network for WAN connectivity

- vmnet2: LAN segment (Kali Linux)

- vmnet3: OPT1 segment (Ubuntu Server)


| Network | Subnet | DHCP Range | Gateway |
|---------|--------|-----------|---------|
| LAN (vmnet2) | 192.168.10.0/24 | 192.168.10.100-200 | pfSense LAN interface |
| OPT1 (vmnet3) | 192.168.20.0/24 | 192.168.20.100-200 | pfSense OPT1 interface |
| WAN (vmnet0) | DHCP from host | N/A | Host network gateway |


### Virtual Machines

**pfSense**: Firewall/router with WAN, LAN, and OPT1 interfaces

**Kali Linux**: Connected to vmnet2 (LAN)

**Ubuntu Server**: Connected to vmnet3 (OPT1)



---

### **Configuration Steps**

**1. Initial Baseline State**

- LAN and OPT1 isolated from WAN

- Inter-subnet routing functional

- No outbound internet access configured


**2. WAN Interface Configuration**

- Changed WAN interface from vmnet1 to vmnet8 (initial attempt)

- Migrated to vmnet8 for proper connectivity


**3. NAT Configuration**

**Navigation**: pfSense → Firewall → NAT → Outbound

- Set outbound NAT to **Automatic outbound NAT rule generation**

- Enables automatic creation of NAT rules for internal networks


**4. Firewall Rules Configuration**

**Initial Configuration (Any Protocol)**

**LAN Rules (vmnet2)**

**Navigation**: pfSense → Firewall → Rules → LAN

- Source: LAN subnet

- Destination: Any

- Protocol: Any

- Action: Allow


**OPT1 Rules (vmnet3)**

**Navigation**: pfSense → Firewall → Rules → OPT1

- Source: OPT1 subnet

- Destination: Any

- Protocol: Any (including DNS on port 53)

- Action: Allow


### Refined Configuration (Protocol-Specific Rules)

After establishing connectivity, the firewall rules were refined to follow security best practices by specifying exact protocols 
and ports instead of allowing all traffic.

**LAN Rules (vmnet2)**

**Navigation**: pfSense → Firewall → Rules → LAN

**Rule 1**:

- Source: LAN subnet

- Destination: Any

- Protocol: TCP/UDP

- Port: 443 (HTTPS)

- Action: Allow


**Rule 2**:

- Source: LAN subnet

- Destination: Any

- Protocol: TCP/UDP

- Port: 80 (HTTP)

- Action: Allow


**Rule 3**:

- Source: LAN subnet

- Destination: Any

- Protocol: TCP/UDP

- Port: 53 (DNS)

- Action: Allow


**Rule 4**:

- Source: LAN subnet

- Destination: Any

- Protocol: ICMP

- Action: Allow


**OPT1 Rules (vmnet3)**

**Navigation**: pfSense → Firewall → Rules → OPT1

**Rule 1**:

- Source: OPT1 subnet

- Destination: Any

- Protocol: TCP/UDP

- Port: 443 (HTTPS)

- Action: Allow


**Rule 2**:

- Source: OPT1 subnet

- Destination: Any

- Protocol: TCP/UDP

- Port: 80 (HTTP)

- Action: Allow


**Rule 3**:

- Source: OPT1 subnet

- Destination: Any

- Protocol: TCP/UDP

- Port: 53 (DNS)

- Action: Allow


**Rule 4**:

- Source: OPT1 subnet

- Destination: Any

- Protocol: ICMP

- Action: Allow


**5. DHCP Server Configuration**

**Navigation**: pfSense → Services → DHCP Server → LAN

- Enabled DHCP server on LAN interface

- DHCP range: 192.168.10.100-192.168.10.200

- Ensures VMs in LAN interface receive proper IP assignment


**Navigation**: pfSense → Services → DHCP Server → OPT1

- Enabled DHCP server on OPT1 interface

- DHCP range: 192.168.20.100 - 192.168.20.200

- Ensures Ubuntu Server receives automatic IP assignment



---

### Troubleshooting Log

#### Issue 1: No Internet Access After Initial Configuration

**Symptoms**: Kali Linux unable to access the internet despite NAT and firewall rules configured

**Investigation**:

- Reviewed firewall rules - confirmed correct

- Checked NIC assignment to vmnet8 - no issues

- Checked interface status: pfSense → Status → Interfaces


**Root Cause**: WAN interface had no IPv4 address or gateway

**Resolution**:

1. Checked Virtual Network Editor - vmnet8 was not created


2. Restored default settings and switched WAN to vmnet8


3. Configured pfSense VM settings: **Settings → Network Adapter → vmnet8**


4. Verified interface now shows IPv4 address and gateway



#### **Issue 2: Unable to Access pfSense Web GUI from Host**

**Symptoms**: Could not connect to pfSense web interface from host machine

**Workaround**: Accessed web GUI through Kali Linux VM after switching it to the LAN


#### Issue 3: Persistent Internet Connectivity Issues

**Symptoms**: Even after network adapter changes, still no internet access (ping 8.8.8.8 failed)

**Troubleshooting Steps**:

1. Reset pfSense firewall to default settings


2. Recreated firewall rules - still no connectivity



####Issue 4: "Connect Host Virtual Adapter" Blocking Internet

**Root Cause**: The "Connect a host virtual adapter to this network" option was enabled on virtual networks, preventing
internet connectivity

**Resolution** (Critical):

1. Opened Virtual Network Editor


2. Disabled "Connect a host virtual adapter to this network" **on both vmnet2 and vmnet3**


3. Tested on Kali Linux - **YouTube successfully opened**



> **Note**: This was the primary issue preventing internet access across all VMs




#### Issue 5: Ubuntu Server No IP Address

**Symptoms**: Ubuntu Server had no IPv4 address assignment

**Investigation**:

- Ran ip a command - no address shown

- Checked Virtual Network Editor - DHCP was disabled on vmnet3

- Noted that vmnet2 also had no DHCP but Kali worked (likely using static IP)


**Resolution**:

1. Enabled DHCP on vmnet3 with range 192.168.20.100-200


2. Still no connectivity - required additional pfSense configuration


3. Disabled DHCP on vmnet3


4. Configured DHCP server on pfSense, OPT1 interface with range 192.168.20.100-200



#### Issue 6: Ubuntu DNS and Protocol Issues

Symptoms:

- ping -c 3 8.8.8.8 worked (ICMP)

- ping [google.com](http://google.com) failed (DNS resolution)

- curl https://google.com failed


**Root Cause**: Missing DNS firewall rule (port 53) and incomplete protocol rules on OPT1

**Resolution**:

1. Added DNS rule (port 53) to OPT1 firewall to allow DNS requests


2. Ubuntu received IPv4 address via ip a


3. Added "any protocol" rule to OPT1 firewall


4. Tested ping [google.com](http://google.com) and curl https://google.com - both successful


5. Refined rules to specify exact protocols needed instead of "any"




---

### Final Working Configuration

**Virtual Network Editor Settings**

- **vmnet8**: NAT mode

- **vmnet2**: Host-only, DHCP disabled, "Connect host virtual adapter" = **OFF**

- **vmnet3**: Host-only, "Connect host virtual adapter" = **OFF**


### pfSense Configuration Summary

**WAN Interface**:

- Connected to vmnet8

- Receives IP via DHCP from host network

- Gateway configured automatically


**LAN Interface (vmnet2)**:

- Static IP configuration (192.168.10.1)

- DHCP server enabled: 192.168.10.100-200

- Firewall rules: Allow all outbound traffic


**OPT1 Interface (vmnet3)**:

- Static IP configuration (192.168.20.1)

- DHCP server enabled: 192.168.20.100-200

- Firewall rules: Specific firewall rules


**NAT**:

- Automatic outbound NAT rule generation enabled

- All internal subnets masquerade through WAN interface


**Connectivity Status**

✅ **Kali Linux (LAN)**: Full internet access via DHCP

✅ **Ubuntu Server (OPT1)**: Full internet access via DHCP

✅ **Inter-subnet routing**: Functional

✅ **Internet connectivity**: Both VMs can reach external resources


---

### Key Lessons Learned

1. **Virtual Network Adapter Settings** The "Connect a host virtual adapter to this network" option can block internet connectivity
   and should be disabled for guest-only networks


3. **DHCP Configuration**: Ensure both VMware DHCP (if needed) and pfSense DHCP services are properly configured for client networks


4. **DNS Requirements**: DNS resolution requires explicit firewall rules allowing UDP/TCP port 53 if no 'any' rule


5. **Troubleshooting Approach**: Check interface status (Status → Interfaces) to verify IP address and gateway assignment before
   investigating complex firewall issues


7. **Protocol-Specific Rules**: While "any protocol" rules work, it's best practice to specify exact protocols (TCP, UDP, ICMP) for
   security purposes




---

### Future Considerations

- Implement more granular firewall rules instead of "allow any"

- Configure VLANs for additional network segmentation

- Set up logging and monitoring for security events

- Document static IP assignments if used

- Consider implementing IDS/IPS capabilities

- Consider configuring a Client to Site VPN

---

### Commands Reference

**Ubuntu Server Commands Used**

**Check IP address configuration**
ip a  
  
**Test ICMP connectivity**  
ping -c 3 8.8.8.8  
  
**Test DNS resolution**  
ping google.com  
  
**Test HTTPS connectivity**  
curl https://google.com

**pfSense Interface Status Check**

Navigate to: **Status → Interfaces**

Verify:

- IPv4 address assigned

- Gateway configuration

- Interface status (up/down)



---

End of Technical Documentation
