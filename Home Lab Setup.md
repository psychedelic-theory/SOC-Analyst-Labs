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

### Screenshots
> <img width="1781" height="1198" alt="image" src="https://github.com/user-attachments/assets/239c29c2-f0e9-48a7-abd1-70ce77574f39" /> [Fig 2 - pfSense Download Page]
> 

