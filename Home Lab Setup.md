# Home Lab Setup
## Objective

The goal of this home lab is to build a fully functional, enterprise-simulating cybersecurity environment from the ground up. By configuring network infratstructure, deploying operating systems, and integrating security tooling, this lab serves as a hands-on foundation for developing real-world SOC Analyst skills.

This lab covers the full setup lifecycle - from provisioning a pfSense firewall and virtualized Windows and Linux machines, to standing a Windows Server with Active Directory for user, group, and policy management, domain joining endpoints, and deploying Sysmon and Splunk for endpoint telemetry and log aggregation. 

The completed environemtn will simulate the kind of network an analyst would monitor and defend in a professional SOC setting, providing a controlled space to practice threat detection, log analysis, and incident response workflows. 

### Progress Tracker
 
| Component | Status |
|---|---|
| Install & Configure pfSense | Not Started |
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

## Installing & Configuring pfSense

### Overview
pfsense acts as the firewall and router for the lab network, segmenting traffic between machines and simulating an enterprise boundary.

### Steps
- [x] Download pfsense ISO [Fig 2]
- [x] Create pfsense VM in VirtualBox/VMware
- [ ] Configure WAN and LAN interfaces
- [ ] Set up DHCP for the lab network
- [ ] Verify connectivity between VMs

## Notes
 During the pfSense installation in Virtualbox I encountered the VM hanging or failing to detect network interfaces. This failure typically surfaced during the install or after the first boot, with interfaces failling to initialize or assign properly. [Fig 8] The issue was having the network adapter type set to Paravirtualized Network (virtio). 
 
 Paravirtualized adapter generally are preferred because they have a higher performance by bypassing emulation overhead, allowing the guest OS to communicate directly with the hypervisor, lower CPU usage, and better throughput. The combination of VirtualBox, FreeBSD/pfSense introduces instability because the virtio driver stability is dependent on the kernel version in use and a history of Oracle VM VirtualBOX occasional virtio regressions and compatibility inconsistencies with BSD guests. 
 
 To remedy the issue, I changed the adatper type to Intel PRO/1000 MT Desktop for each interface. The tradeoff was for stability over performance. 

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
> 



