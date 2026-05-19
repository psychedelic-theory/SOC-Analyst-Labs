# Home Lab Setup
## Objective

The goal of this home lab is to build a fully functional, enterprise-simulating cybersecurity environment from the ground up. By configuring network infratstructure, deploying operating systems, and integrating security tooling, this lab serves as a hands-on foundation for developing real-world SOC Analyst skills.

This lab covers the full setup lifecycle - from provisioning a pfSense firewall and virtualized Windows and Linux machines, to standing a Windows Server with Active Directory for user, group, and policy management, domain joining endpoints, and deploying Sysmon and Splunk for endpoint telemetry and log aggregation. 

The completed environemtn will simulate the kind of network an analyst would monitor and defend in a professional SOC setting, providing a controlled space to practice threat detection, log analysis, and incident response workflows. 

### Progress Tracker
 
| Component | Status |
|---|---|
| Install & Configure pfSense | Complete |
| Install Windows 11 VM | Not Started |
| Download & Configure Kali Linux VM | Not Started |
| Install Windows Server | Not Started |
| Install & Configure Active Directory | Not Started |
| Manage Users, Groups & Policies | Not Started |
| Domain Joining | Not Started |
| Install Sysmon | Not Started |
| Install & Configure Splunk | Not Started |

## Lab Network Diagram
<img width="2324" height="1888" alt="image" src="https://github.com/user-attachments/assets/1a2a4d52-8e9f-4a85-af9e-2a0f645e6c50" /> [Fig 1 - Network Topology]

# **Installing & Configuring pfSense**

### Overview
pfsense acts as the firewall and router for the lab network, segmenting traffic between machines and simulating an enterprise boundary.

### Steps
- [x] Download pfsense ISO [Fig 2]
- [x] Create pfsense VM in VirtualBox/VMware
- [ ] Configure WAN and LAN interfaces
- [ ] Set up DHCP for the lab network
- [ ] Verify connectivity between VMs

## Notes
### Paravirtualized Connectivity Issue
 During the pfSense installation in Virtualbox I encountered the VM hanging or failing to detect network interfaces. This failure typically surfaced during the install or after the first boot, with interfaces failling to initialize or assign properly. [Fig 8] The issue was having the network adapter type set to Paravirtualized Network (virtio). 
 
 Paravirtualized adapter generally are preferred because they have a higher performance by bypassing emulation overhead, allowing the guest OS to communicate directly with the hypervisor, lower CPU usage, and better throughput. The combination of VirtualBox, FreeBSD/pfSense introduces instability because the virtio driver stability is dependent on the kernel version in use and a history of Oracle VM VirtualBOX occasional virtio regressions and compatibility inconsistencies with BSD guests. 
 
 To remedy the issue, I changed the adatper type to Intel PRO/1000 MT Desktop for each interface. The tradeoff was for stability over performance. 
 ### WAN Adapter Atachment Issue
 I encountered another warning stating that the installer could not reach the Netgate servers during the installation process. The lab was designed to mimic a realistic enterprise environment with pfSense acting as the central firewall and router between three network segments (WAN, LAN 0, & Lan 1). 

 The WAN adapter (Fig 5) was configured as a Bridged Adapter connected to the host machine's Intel WI-FI 7 BE201 wireleess NIC. This introduced a ccommon VirtualBox limitation where bridged networking over a wireless adapter is inherently unreliable. Wi-Fi NICs operate at Layer 2 in a way that most home routers and wireless access points cannot or do not support. Home routers for example typically will not forward traffic destined for a MAC address that differs from the registered host machine. With out pfSense machine having its own unique MAC address, the router silently dropped/ignored the traffic. 

 To resolve the issue, we switched Adapter 1 (Fig 16) from Bridged Adapter to NAT, because NAT does not require selecting a specific host NIC. This allows pfSense to reach the internet without needing to interact with the home router directly, completly bypassing the Wi-Fi bridging limitation.

 

### Screenshots
> <img width="1781" height="1198" alt="image" src="https://github.com/user-attachments/assets/239c29c2-f0e9-48a7-abd1-70ce77574f39" /> [Fig 2 - pfSense Download Page]
> <img width="1910" height="1098" alt="image" src="https://github.com/user-attachments/assets/4707bcc1-a2c0-44ec-8995-efcc190306da" /> [Fig 3a - Configuring pfSense in VirtualBox]
> <img width="1911" height="1105" alt="image" src="https://github.com/user-attachments/assets/d8fe6ab6-f7ed-4887-a3de-0b36343852b4" /> [Fig 3b - Configuring pfSense in VirtualBox]
> <img width="1908" height="1095" alt="image" src="https://github.com/user-attachments/assets/89d7afff-dc62-4b0e-a21d-1f507125ef5e" /> [Fig 3c - Configuring pfSense in VirtualBox]
> <img width="1114" height="877" alt="image" src="https://github.com/user-attachments/assets/1054fe53-2eb3-499a-972b-0564d36b2397" /> [Fig 4 - pfSense System Settings]
> <img width="1119" height="534" alt="image" src="https://github.com/user-attachments/assets/1a2c8e50-5c13-4cd9-938d-0a1900095299" /> [Fig 5 - Network Adapter 1]
> <img width="1126" height="550" alt="image" src="https://github.com/user-attachments/assets/bfc48185-833a-467c-8e7d-57c32427d895" /> [Fig 6 - Network Adapter 2]
> <img width="1122" height="547" alt="image" src="https://github.com/user-attachments/assets/a3345aec-aba0-4196-b847-23bd8a28a3a2" /> [Fig 7 - Network Adapter 3]
> <img width="722" height="566" alt="image" src="https://github.com/user-attachments/assets/51b34d8a-31e6-4c90-92ae-4a2867b4ab9c" /> [Fig 8 - pfSense installation Failure]
> <img width="712" height="562" alt="image" src="https://github.com/user-attachments/assets/a3c2eaf9-0d8d-4d8e-9cab-73023c4541b9" /> [Fig 9 - Accept Installation]
>  <img width="719" height="563" alt="image" src="https://github.com/user-attachments/assets/b2b7cbc8-caa1-47d8-805c-038b2e782c7b" /> [Fig 10 - pfSense Installer]
> <img width="720" height="567" alt="image" src="https://github.com/user-attachments/assets/189364fd-dbbb-4ad0-826a-c17dd18797b0" /> [Fig 11 - pfSense WAN Interface Setup]
> <img width="723" height="566" alt="image" src="https://github.com/user-attachments/assets/fe3303d9-938d-465c-b559-c11784a1a947" /> [Fig 12 - Install CE]
> <img width="710" height="561" alt="image" src="https://github.com/user-attachments/assets/1329a257-b72e-441e-8194-c0c488cb0aaa" /> [Fig 13 - Finalizing Installation]
> <img width="1127" height="390" alt="image" src="https://github.com/user-attachments/assets/fefd7eef-947d-4abd-bcc4-ea88ea5bf0c6" /> [Fig 14 - Removing Optical Drive]
> <img width="1123" height="603" alt="image" src="https://github.com/user-attachments/assets/d1eaadad-5be6-467e-8414-4d9370ba87dc" /> [Fig 15 - Temporarily Changing Adapter 1 to NAT]
> <img width="713" height="570" alt="image" src="https://github.com/user-attachments/assets/fb2785e2-e587-41b0-a9e0-cc5e86f99066" /> [Fig 16 - pfSense Menu]
> <img width="718" height="403" alt="image" src="https://github.com/user-attachments/assets/9899ae9b-07f6-4199-81ea-37152f9db874" /> [Fig 17a - Assigning Interfaces]
> <img width="720" height="405" alt="image" src="https://github.com/user-attachments/assets/22c17416-6d0f-4b71-aed0-9065b9f48ab5" /> [Fig 17b - Assigning Interfaces]
> <img width="724" height="354" alt="image" src="https://github.com/user-attachments/assets/087e25dd-c4ea-416c-b380-253f08fb1734" /> [Fig 18a - LAN Interface Configuration]
> <img width="722" height="244" alt="image" src="https://github.com/user-attachments/assets/4bb87e7b-aed6-482d-aeaa-19eedce4172e" /> [Fig 18b - LAN Interface Configuration]
> <img width="724" height="399" alt="image" src="https://github.com/user-attachments/assets/8ceb62c7-082e-446b-abf5-d643e77b99eb" /> [Fig 18c - LAN Interface Configuration]
> <img width="720" height="355" alt="image" src="https://github.com/user-attachments/assets/e789ffa3-8227-430d-bd3b-90c74a1d4fec" /> [Fig 19a - OPT1 Interface Configuration]
> <img width="705" height="245" alt="image" src="https://github.com/user-attachments/assets/d54c1a57-ce59-48b0-8000-575b9486839f" /> [Fig 19b - OPT1 Interface Configuration]
> <img width="727" height="217" alt="image" src="https://github.com/user-attachments/assets/c7b3fcdf-3bb9-4323-a5f4-030b730572c6" /> [Fig 19c - OPT1 Interface Configuration]
> <img width="727" height="410" alt="image" src="https://github.com/user-attachments/assets/b5cb8c52-5612-46e1-9534-097d6ea3a831" /> [Fig 20 - pfSense Final Configuration]







