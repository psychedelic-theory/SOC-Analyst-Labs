# Home Lab Setup
## Objective

The goal of this home lab is to build a fully functional, enterprise-simulating cybersecurity environment from the ground up. By configuring network infratstructure, deploying operating systems, and integrating security tooling, this lab serves as a hands-on foundation for developing real-world SOC Analyst skills.

This lab covers the full setup lifecycle - from provisioning a pfSense firewall and virtualized Windows and Linux machines, to standing a Windows Server with Active Directory for user, group, and policy management, domain joining endpoints, and deploying Sysmon and Splunk for endpoint telemetry and log aggregation. 

The completed environemtn will simulate the kind of network an analyst would monitor and defend in a professional SOC setting, providing a controlled space to practice threat detection, log analysis, and incident response workflows. 

### Progress Tracker
 
| Component | Status | 
|---|---|
| [Install & Configure pfSense Part 1](#pfsense) | Complete |
| [Configure pfSense Part 2](#pfSense2) | Complete |
| [Install Windows 11 VM](#win11) | Complete |
| [Download & Configure Kali Linux VM](#kali) | Complete |
| [Install Windows Server](#wser) | In Progress |
| Install & Configure Active Directory | Not Started |
| Manage Users, Groups & Policies | Not Started |
| Domain Joining | Not Started |
| Install Sysmon | Not Started |
| Install & Configure Splunk | Not Started |

## Lab Network Diagram
<img width="2324" height="1888" alt="image" src="https://github.com/user-attachments/assets/1a2a4d52-8e9f-4a85-af9e-2a0f645e6c50" /> [Fig 1 - Network Topology]

# <a name="pfsense"></a>Installing & Configuring pfSense Part 1

### Overview
pfsense acts as the firewall and router for the lab network, segmenting traffic between machines and simulating an enterprise boundary.

### Steps
- [x] Download pfsense ISO [Fig 2]
- [x] Create pfsense VM in VirtualBox/VMware
- [x] Configure WAN and LAN interfaces
- [X] Set up DHCP for the lab network

### Notes
#### Paravirtualized Connectivity Issue
 During the pfSense installation in Virtualbox I encountered the VM hanging or failing to detect network interfaces. This failure typically surfaced during the install or after the first boot, with interfaces failling to initialize or assign properly. [Fig 8] The issue was having the network adapter type set to Paravirtualized Network (virtio). 
 
 Paravirtualized adapter generally are preferred because they have a higher performance by bypassing emulation overhead, allowing the guest OS to communicate directly with the hypervisor, lower CPU usage, and better throughput. The combination of VirtualBox, FreeBSD/pfSense introduces instability because the virtio driver stability is dependent on the kernel version in use and a history of Oracle VM VirtualBOX occasional virtio regressions and compatibility inconsistencies with BSD guests. 
 
 To remedy the issue, I changed the adatper type to Intel PRO/1000 MT Desktop for each interface. The tradeoff was for stability over performance. 
 #### WAN Adapter Atachment Issue
 I encountered another warning stating that the installer could not reach the Netgate servers during the installation process. The lab was designed to mimic a realistic enterprise environment with pfSense acting as the central firewall and router between three network segments (WAN, LAN 0, & Lan 1). 

 The WAN adapter (Fig 5) was configured as a Bridged Adapter connected to the host machine's Intel WI-FI 7 BE201 wireleess NIC. This introduced a ccommon VirtualBox limitation where bridged networking over a wireless adapter is inherently unreliable. Wi-Fi NICs operate at Layer 2 in a way that most home routers and wireless access points cannot or do not support. Home routers for example typically will not forward traffic destined for a MAC address that differs from the registered host machine. With out pfSense machine having its own unique MAC address, the router silently dropped/ignored the traffic. 

 To resolve the issue, we switched Adapter 1 (Fig 16) from Bridged Adapter to NAT, because NAT does not require selecting a specific host NIC. This allows pfSense to reach the internet without needing to interact with the home router directly, completly bypassing the Wi-Fi bridging limitation.
#### WAN & LAN Configuration
The WAN interface (em0) received a DHCP lease of 10.0.2.15/24 from VirutalBox's NAT engine, confirming that the earlier adapter change from Bridge to NAT was successful. pfSense now has reliable outbound internet access routed through the host machine without depending on Wi-Fi bridging.

The Lan interface was assigned 10.0.1.1/24, serving as the default gateway for the ECorp network segment. The /24 subnet provides 254 usable addresses (10.0.1.1 - 10.0.1.254), with pfSense holding .1 as the gateway. The remaining address space is deliberately left available to accomodate future static IP assignments for infrastructure such as domain controllers, file servers, SIEM nodes, or any additional lab machines that require a fixed, predictable address.

OPT1 was assigned 10.0.3.1/24 as the gatewat for the Attack LAN segment following the same subnet logic - pfSense anchors the .1 address and the remaining range is available for the Kali Linux machine and any future attack infrastructure added to the lab.

The webConfigurator is accessible at https://10.0.3.1/ for GUI-based firewall management and further configuration.

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

# <a name="pfsense2"></a>Configuring pfSense Part 2

### Overview
This section covers the initial web-based configuration of pfSense after installation. Using the Windows 11 VM as your management station, you access the pfSense web GUI, run through the setup wizard, name and organize your network interfaces, configure DNS and DHCP services, assign a static IP to the Windows 11 machine, and build out a structured set of firewall rules to control traffic between your network segments (ECorp LAN, Attack LAN, and the internet).
By the end of this section, pfSense is fully configured as the segmented firewall and router for the lab environment.

### Steps
- [X] Access pfSense Web GUI
- [X] Run the Setup Wizard
- [X] Name the interfaces
- [X] Configure DNS Resolver
- [X] Enable Hardware Checksum Offloading
- [X] Assign a Static DHCP Lease to Windows 11
- [X] Set DNS Servers for DHCP
- [X] Create a Firewall Alias
- [X] Create ECorp Firewall Rules
- [X] Create AttackLAN Firewall Rules
- [X] Verify Connectivity (Optional)
- [X] Reboot pfSense

### Notes
#### Setup Wizard
Unchecking "Override DNS" on the General Information screen matters because leaving it enabled would allow DHCP/PPP on the WAN to override the DNS settings you configure, which can cause inconsistent name resolution behavior across the lab.

Unchecking "Block RFC1918 Private Networks" on the WAN interface VirtualBox NAT, the WAN-side IP is itself a private address ('10.0.2.15'). Leaving this option checked would cause pfSense to silently drop its own upstream traffic, breaking internet access entirely. this is a common point of confusion when running pfSense in a virtualized environment rather than on physical hardware where the WAN would normally face a public IP.

#### DNS Resolver Configuration
Enabling DHCP Registration and Static DHCP in the DNS Resolver ensures that machines on the network can resolve each other by hostname rather than just IP address. This becomes important later when domain joining and Active Directory are introduced - hostname resolution needs to be reliabnle before those components will function properly. 

Enabling Prefetch Support and Prefetch DNS Key Support in the Advanced Settings keeps DNS performant by refreshing cache entries before they expire and reducing DNSSEC validation latency. These are small optimizations, but they reflect good practice for a lab simulating an enterprise environment.

#### Hardware Checksum Offloading
Disabling hardware checksum offloading under `System → Advanced → Networking` is a VirtualBox-specific requirement. VirtualBox's virtual NICs do not reliably handle checksum offloading, which can cause packet corruption or intermittent connectivity failures that are difficult to diagnose. The symptoms can look like a firewall or routing issue when the real cause is at a much lower level. Disabling it forces checksums to be calculated in software, which is slower in theory but far more stable in this environment. This is a known quirk of running pfSense under VirtualBox and worth documenting to avoid chasing phantom network issues later.

#### Firewall Alias (RFC1918) 
Creating an alias that groups the three private IPv4 ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) into a single named object called `RFC1918` is a small step that significantly improves firewall rule readability. Rather than writing three separate destination entries every time a rule needs to reference private address space, a single alias reference handles it. This is standard practice in production firewall management and keeps the rule base clean and maintainable as the lab grows.

#### General Observations
 
A few patterns worth carrying forward from this section:
 
**Apply Changes is not optional.** pfSense separates saving a configuration from activating it. A saved rule that has not been applied is not running. This is easy to forget mid-workflow when moving quickly between settings.
 
**Order matters in firewall rules.** pfSense evaluates rules top to bottom and stops at the first match. The block-all rule at the bottom of the ECorp ruleset only works as a catch-all because the more specific pass rules sit above it. Placing rules in the wrong order can silently allow or deny traffic in unexpected ways.
 
**Both DHCP scopes need attention.** Any configuration applied to the ECORP DHCP scope (DNS servers, options) usually also needs to be applied to the ATTACKLAN scope. It is a consistent source of asymmetric behavior when one interface is configured and the other is forgotten.


### Screenshots
> <img width="1422" height="932" alt="image" src="https://github.com/user-attachments/assets/20a3b2de-7e65-4226-b577-cab777205a17" /> [Fig 1 - pfSense's Self-Signed Certificate Warning]
> <img width="1424" height="931" alt="image" src="https://github.com/user-attachments/assets/c4891529-52cc-4e28-b0e0-fbb7e9592984" /> [Fig 2 - pfSense Web GUI]
> <img width="1422" height="926" alt="image" src="https://github.com/user-attachments/assets/607a42c7-48e0-45c7-a39e-e866e06b5b78" /> [Fig 3 - pfsense GUI Setup]
> <img width="1425" height="930" alt="image" src="https://github.com/user-attachments/assets/e17b74e7-ab92-4abc-8a4f-511dbeeb06e8" /> [Fig 4 - pfSense GUI Netgate]
> <img width="1422" height="930" alt="image" src="https://github.com/user-attachments/assets/2f85f23a-46f1-4e6a-8387-6e1d5c008952" /> [Fig 5 - Password Change]
> <img width="1418" height="925" alt="image" src="https://github.com/user-attachments/assets/dcfe0262-7c64-4531-b5c7-e91282adfe0b" /> [Fig 6 - Domain Name Change]
> <img width="1430" height="932" alt="image" src="https://github.com/user-attachments/assets/6aa98ce8-e07b-4308-a6bc-dc495a05beff" /> [Fig 7 - Time Server Information]
> <img width="1420" height="931" alt="image" src="https://github.com/user-attachments/assets/c1b020e8-7514-4385-b84d-e5fea0ab1afc" /> [Fig 8 - WAN Interface Configuration]
> <img width="1423" height="933" alt="image" src="https://github.com/user-attachments/assets/7f7105e8-bdf6-4a26-bffb-8d284eba0ac2" /> [Fig 9 - LAN Interface Configuration]
> <img width="1422" height="932" alt="image" src="https://github.com/user-attachments/assets/39a53730-dcf8-4290-b9a4-d1582d56b4ce" /> [Fig 10 - Change Admin Account Password]
> <img width="1420" height="929" alt="image" src="https://github.com/user-attachments/assets/56dd72f5-5248-4bb4-b22a-a31d67369219" /> [Fig 11 - Reload Wizard Configuration]
> <img width="1419" height="925" alt="image" src="https://github.com/user-attachments/assets/97ffd2f7-d132-4dd7-965b-79ee2e8ab146" /> [Fig 12 - Wizard Setup Complete]
> <img width="1418" height="926" alt="image" src="https://github.com/user-attachments/assets/dd69652f-325f-4df1-a134-a499dce400ac" /> [Fig 13 - Copyright and Trademark]
> <img width="1430" height="902" alt="image" src="https://github.com/user-attachments/assets/abaa8ad0-67c2-4c33-94c7-0946cc07a675" /> [Fig 14 - Ecorp LAN Interface]
> <img width="1440" height="902" alt="image" src="https://github.com/user-attachments/assets/17162959-1230-4a32-b631-6d7bd8390349" /> [Fig 15 - Attack LAN Interface]
> <img width="1439" height="903" alt="image" src="https://github.com/user-attachments/assets/3496b35c-1ab5-4e00-a2cd-d913345d5b72" /> [Fig 16a - DNS Resolver General Settings]
> <img width="1436" height="899" alt="image" src="https://github.com/user-attachments/assets/16151617-cd6e-4fd4-9f14-da24a5a7c99d" /> [Fig 16b - DNS Resolver Advanced Settings]
> <img width="1438" height="898" alt="image" src="https://github.com/user-attachments/assets/32760b64-7338-45fb-8b9b-362473998fd8" /> [Fig 17 - System/Advanced/Networking]
> <img width="1432" height="903" alt="image" src="https://github.com/user-attachments/assets/71e707ef-9cce-4a83-b73b-55444b7e12fe" /> [Fig 18 - Reboot Hardware Checksum Setting]
> <img width="1432" height="896" alt="image" src="https://github.com/user-attachments/assets/578ac987-0266-4ec0-97aa-0c5a8259d912" /> [Fig 19 - Reboot Page]
> <img width="1439" height="901" alt="image" src="https://github.com/user-attachments/assets/1a76c719-7908-43b4-b14b-ad74f3ad6bd0" /> [Fig 20 - DHCP Leases]
> <img width="1435" height="1530" alt="image" src="https://github.com/user-attachments/assets/238865f1-2ea0-478b-8cb5-a6a15efe691a" /> [Fig 21 - Win VM Static Mapping Configuration]
> <img width="1107" height="800" alt="image" src="https://github.com/user-attachments/assets/53469801-e138-483a-ac88-97c0d54e4bdc" /> [Fig 22 - Successful Static Mapping]
> <img width="1437" height="1450" alt="image" src="https://github.com/user-attachments/assets/dc9099f1-a37e-42f8-bfed-ae58f94ec95b" /> [Fig 23a - ECORP DHCP Server]
> <img width="1432" height="1485" alt="image" src="https://github.com/user-attachments/assets/10d91049-e69f-472b-b82e-a17c359a0bf3" /> [Fig 23b - AttackLAN DHCP Server]
> <img width="1436" height="918" alt="image" src="https://github.com/user-attachments/assets/0a28aa0d-36bc-4b29-a7ce-e152e1bfedad" /> [Fig 24a - Firewall Aliases]
> <img width="1437" height="1061" alt="image" src="https://github.com/user-attachments/assets/8c330fab-a1fc-418d-a5f9-adbb4447aad0" /> [Fig 25a - Ecorp Firewall Rule For Device Comm.]
> <img width="1442" height="1062" alt="image" src="https://github.com/user-attachments/assets/cce81c12-4b6b-4a00-bd11-4f1207bdd018" /> [Fig 25b - Firewall Rule Comm. b/w Ecorp & Attack LAN]
> <img width="1438" height="1059" alt="image" src="https://github.com/user-attachments/assets/af813195-c60b-4be0-b819-366782f80264" /> [Fig 25c - Firewall Rule Three]
> <img width="1452" height="1058" alt="image" src="https://github.com/user-attachments/assets/55590b28-1871-4396-88f9-90bab53ee50b" /> [Fig 25d - Firewall Rule Blocks Anything Prev. Unspecified]
> <img width="1439" height="929" alt="image" src="https://github.com/user-attachments/assets/1e354184-1441-4ae8-ac7b-716c36085c5a" /> [Fig 25e - ECORP Firewall Rules]
> <img width="1443" height="1058" alt="image" src="https://github.com/user-attachments/assets/e113c728-cb9c-4cfa-8849-43d2cf893f4c" /> [Fig 26a - AttackLAN Firewall Rule ~ Traffic from AL to ECorp]
> <img width="1441" height="1060" alt="image" src="https://github.com/user-attachments/assets/e6045ddc-2034-4382-86a2-85a187e40417" /> [Fig 26b - AttackLAN Firewall Rule ~ Traffic to Internet]
> <img width="1435" height="927" alt="image" src="https://github.com/user-attachments/assets/0f6995a3-dbb5-44e2-982f-938e4a8fd338" /> [Fig 26c - AttackLAN Firewall Rules]
> <img width="1446" height="453" alt="image" src="https://github.com/user-attachments/assets/e0e3dc0a-e214-47ed-973e-8fb63b8655fd" /> [Fig 27 - Diagnostics/Reboot]

# <a name="win11"></a>Installing Windows 11 VM

### Overview
Installation and configuration of a Windows 11 virtual machine to serve as a workstation endpoint in the Ecorp network segment.

### Steps
- [x] Download Windows 11 ISO
- [x] Create Windows 11 VM in VirtualBox
- [x] Configure network adapter to LAN 0
- [x] Complete initial Windows setup

### Screenshots
> <img width="1400" height="483" alt="image" src="https://github.com/user-attachments/assets/cd8855d5-00cc-4834-a36c-4756ebcbdca4" /> [Fig 1 - Download Win11 ISO]
> <img width="2124" height="737" alt="image" src="https://github.com/user-attachments/assets/857c22f4-54a9-4f22-897e-d83886c17bf6" /> [Fig 2a - Win11 VM OS Setup]
> <img width="2120" height="647" alt="image" src="https://github.com/user-attachments/assets/71d3931c-9d22-4e61-93fe-fc7d1c7c6730" /> [Fig 2b - Win11 Virtual Hardware Setup]
> <img width="2115" height="935" alt="image" src="https://github.com/user-attachments/assets/72f17ee0-cc61-4421-b12a-b9839335fecb" /> [Fig 2c - Win11 Hard Disk Setup]
> <img width="1147" height="832" alt="image" src="https://github.com/user-attachments/assets/0d6c4072-7da0-4358-827e-b2cd7d7d614b" /> [Fig 3a - Win11 System Settings]
> <img width="1158" height="545" alt="image" src="https://github.com/user-attachments/assets/9b77d027-81ac-4cc3-a6b0-06e6f8b95d4e" /> [Fig 3b - Win11 Network Settings]
> <img width="1202" height="976" alt="image" src="https://github.com/user-attachments/assets/2f8ebf59-2d19-45aa-b6a0-bfa13201abd1" /> [Fig 4a - Install Win11]
> <img width="839" height="656" alt="image" src="https://github.com/user-attachments/assets/58794eb7-eb59-4b7e-b9ae-eaa125e2e0ce" /> [Fig 4b - Windows11 Install Allocation]
> <img width="1069" height="542" alt="image" src="https://github.com/user-attachments/assets/02f23c36-63b9-4a73-840f-71aed384d34e" /> [Fig 4c - Windows11 Installing]
>  <img width="1021" height="763" alt="image" src="https://github.com/user-attachments/assets/ae42e24f-1da4-44a2-b331-7542c488f4a8" /> [Fig 4d - System Reboot]
> <img width="1025" height="767" alt="image" src="https://github.com/user-attachments/assets/50fc952a-6ea4-4353-8d7d-cfb9093c3896" /> [Fig 5a - Windows 11 Region Configuration]
> <img width="1024" height="767" alt="image" src="https://github.com/user-attachments/assets/0c215b18-a868-4d21-be68-e51f774716e7" /> [Fig 5b - Windows 11 Internet Configuration]
> <img width="1025" height="770" alt="image" src="https://github.com/user-attachments/assets/51208f8b-c741-46ef-aa96-d15eed113dc7" /> [Fig 5c - Windows 11 User Creation]
> <img width="1020" height="766" alt="image" src="https://github.com/user-attachments/assets/b331698e-86d3-452a-8db5-cfa75df4a1d7" /> [Fig 5d - Windows 11 Password Creation]
> <img width="1022" height="770" alt="image" src="https://github.com/user-attachments/assets/8dfb9209-12e5-4cc0-bf6d-075b52deb08c" /> [Fig 5e - Windows 11 Password Confirmation]
> <img width="1025" height="766" alt="image" src="https://github.com/user-attachments/assets/add2a6a6-5a4a-40c5-8acb-9302dfffec5a" /> [Fig 5f - Windows Security Questions]
> <img width="1025" height="769" alt="image" src="https://github.com/user-attachments/assets/708fb26c-ff58-4eed-8c05-024718a746bd" /> [Fig 5g - Disable Privacy Settings]
> <img width="1021" height="767" alt="image" src="https://github.com/user-attachments/assets/14db4ea6-c878-40ec-8fb3-8b77c3783410" /> [Fig 5h - Final Restart]
> <img width="1070" height="944" alt="image" src="https://github.com/user-attachments/assets/7bad49fb-af5d-400b-9db4-d41e18d73200" /> [Fig 6a - Insert Guest Additions]
> <img width="1036" height="935" alt="image" src="https://github.com/user-attachments/assets/a1a175bc-de2e-46ee-8333-75c99d64283e" /> [Fig 6b - Download Guest Additions]
> <img width="1027" height="935" alt="image" src="https://github.com/user-attachments/assets/00355270-85e6-4c56-aec3-56ec960c4411" /> [Fig 6.1 - Accept All Necessary Drivers & Software]
> <img width="1023" height="930" alt="image" src="https://github.com/user-attachments/assets/0fd59f9e-122a-42b2-baf0-ca1bfa70d8fd" /> [Fig 6.2 - Install Location]
> <img width="1023" height="928" alt="image" src="https://github.com/user-attachments/assets/4dc34707-5f76-40fb-991c-bf70573cd863" /> [Fig 6.3 - Choose Components]
> <img width="1021" height="934" alt="image" src="https://github.com/user-attachments/assets/7d7b4b50-3945-48cd-a838-634026a668ae" /> [Fig 6.4 - Reboot System]
> <img width="1556" height="1031" alt="image" src="https://github.com/user-attachments/assets/2493f5d1-c87f-47f6-b43a-cfd728d29d71" /> [Fig 7 - Bidirectional Copy/Paste]
> <img width="1112" height="627" alt="image" src="https://github.com/user-attachments/assets/ff4f54f1-6bef-4457-9381-666167f8b191" /> [Fig 8 - ip address & google ping]

# <a name="kali"></a>Download & Configure Kali Linux VM

### Overview
Downloading and importing the pre-built Kali Linux VM into VirtualBox and configuring it to operate on the Attack LAN segment. Unlike the Windows 11 VM which was installed from an ISO, Kali is imported as pre-built virtual machine image - significantly streamlining setup. Once networked and booted, Kali will serve as the attacker machine in the lab, sitting on the OPT1 (Attack LAN) segment isolated from the ECorp network by pfSense firewall rules. 

### Steps 
- [X] Download Kali Linux VM Image From kali.org
- [X] Extract the VM archive using 7-Zip
- [X] open the .vbox file in VirtualBoc
- [X] Configure the network adapter to Internal Network / LAN1 with Paravirtualized Network
- [X] Enable Bidirectional Shared Clipboard and Drag-and-Drop
- [X] Start the VM and log in
- [X] Verify Network Assignment with 'ip a'
- [X] Verify Internet Connectivity with 'ping 8.8.8.8'

### Notes
#### VM Image over Installer Image
Select Virtual Machines and then choose VirtualBox as your hypervisor. The pre-built VM image skips the full OS installation process and drops you directly into a configured Kali environment, which is the faster path for lab purposes. 

#### Default Credentials
Kali's pre-built VM ships with default credentials: username kali, password kali. You will be prompted at the login screen immediately on boot. It is good practice to change these once the VM is up, especially in a lab that simulates a real enterprise network. 

### Screenshots
> <img width="1443" height="1118" alt="image" src="https://github.com/user-attachments/assets/6bb8fb47-ccbd-4f68-92b9-420a2366ab1f" /> [Fig 1 - Kali Landing Page]
> <img width="1438" height="1117" alt="image" src="https://github.com/user-attachments/assets/1daf2006-7344-41fd-a36e-681d730ff67a" /> [Fig 2 - Pre-built VMs]
> <img width="1141" height="935" alt="image" src="https://github.com/user-attachments/assets/d22c0ff3-39dc-4875-a9d2-15aaf5f35a4a" /> [Fig 3 - Extracting Kali VM]
> <img width="1513" height="628" alt="image" src="https://github.com/user-attachments/assets/b784e5c4-b598-486c-b7a1-2e090b8e9308" /> [Fig 4 - Extracted Files]
> <img width="1436" height="899" alt="image" src="https://github.com/user-attachments/assets/b17a3a04-dd3e-4d7d-a505-85c033685ea5" /> [Fig 5 - Kali Linux VM]
> <img width="1114" height="541" alt="image" src="https://github.com/user-attachments/assets/9cbe5592-e3e0-4dbf-8c9e-b4af9cc0a708" /> [Fig 6 - Network Settings]
> <img width="1123" height="427" alt="image" src="https://github.com/user-attachments/assets/5dc8657a-0f8d-4151-b36f-34b346d7f5b1" /> [Fig 7 - Bidirectional Clipping]
> <img width="1286" height="962" alt="image" src="https://github.com/user-attachments/assets/2be69eb2-a0e8-42f5-8c40-a34c66a899a3" /> [Fig 8 - Kali Homepage]
> <img width="648" height="518" alt="image" src="https://github.com/user-attachments/assets/25fe415d-eb44-42f5-898f-2c1037aaac1f" /> [Fig 9 - Connectivity Test]
> <img width="1391" height="734" alt="image" src="https://github.com/user-attachments/assets/60247c15-4afc-458c-89b0-b42a1f27779f" /> [Fig 10 - DHCP Lease Static IP]



# <a name="wser"></a> Installing Windows Server

### Overview
This section covers installing Windows Server 2025 in VirtualBox, which will later be promoted to a Domain Controller (DC-1) for the ECorp network segment. The server is placed on the ECorp LAN (10.0.1.x), assigned a static IP via pfSense, and configured with VirtualBox Guest Additions for improved usability.

### Steps
- [x] Download Windows Server 2025 ISO from Microsoft Evaluation Center
- [x] Create Windows Server VM in VirtualBox
- [x] Configure VM system and network settings
- [x] Install Windows Server 2025 (Desktop Experience)
- [X] Create Administrator password and log in
- [X] Install VirtualBox Guest Additions
- [X] Verify DHCP lease and confirm correct IP range
- [X] Assign static IP (10.0.1.3) via pfSense DHCP static mapping
- [X] Enable bidirectional clipboard between host and VM

### Notes

### Screenshots
> <img width="1415" height="1017" alt="image" src="https://github.com/user-attachments/assets/0478cd3a-5631-4f5d-b5ee-644ea51677bc" /> [Fig 1 - ISO Download]
> <img width="1395" height="507" alt="image" src="https://github.com/user-attachments/assets/1cc4c3ed-6db8-412d-a6a3-116a8d5f4920" /> [Fig 2 - Windows Server 2025]
> <img width="1910" height="1097" alt="image" src="https://github.com/user-attachments/assets/a0b7ebc7-e47e-4e1f-ade7-d6696e27a471" /> [Fig 3a - ECORP DC System Cofiguration]
> <img width="1914" height="1092" alt="image" src="https://github.com/user-attachments/assets/40b27426-70f5-4038-9336-d1bd38347ac9" /> [Fig 3b - Virtual Hardware Configuration]
> <img width="1906" height="1093" alt="image" src="https://github.com/user-attachments/assets/5eecd87f-c3d1-4df4-8d39-2dec0ed22dd5" /> [Fig 3c - Virtual Hard Disk]
> <img width="1555" height="1129" alt="image" src="https://github.com/user-attachments/assets/09838563-cf17-42bc-86da-14c6759a2cee" /> [Fig 3d - System Settings]
> <img width="1111" height="334" alt="image" src="https://github.com/user-attachments/assets/5b0cd6d2-9b3a-4b35-bdac-f6cef66e0852" /> [Fig 3e - Disable Audio]
> <img width="1118" height="545" alt="image" src="https://github.com/user-attachments/assets/d3970e2e-caa0-42e0-9c01-307b1e5ee723" /> [Fig 3f - Network Settings]
> <img width="1027" height="778" alt="image" src="https://github.com/user-attachments/assets/e15237f2-ac02-41f1-b2a2-d80bd753bae1" /> [Fig 4a - Language Selection]
> <img width="1017" height="778" alt="image" src="https://github.com/user-attachments/assets/74de002c-9eb9-4a0f-8adf-38a4d2e49b45" /> [Fig 4b - Setup Option]
> <img width="1021" height="773" alt="image" src="https://github.com/user-attachments/assets/08e79256-8485-4082-94c2-82df5fea1560" /> [Fig 4c - Image Selection]
> <img width="1023" height="769" alt="image" src="https://github.com/user-attachments/assets/a6c8dc21-c6a9-4909-92ca-539e72080694" /> [Fig 4d - Accept Terms & Conditions]
> <img width="1021" height="769" alt="image" src="https://github.com/user-attachments/assets/cf4a0d87-5707-4381-bec6-45048b7d3f24" /> [Fig 4e - Windows Server Allocation]
> <img width="1022" height="773" alt="image" src="https://github.com/user-attachments/assets/7575150c-f8d7-4759-b4b4-266dba0ae5e1" /> [Fig 4f - Install Windows Server]
> <img width="1023" height="771" alt="image" src="https://github.com/user-attachments/assets/d89e0130-b5d9-4621-8710-cb02139d68aa" /> [Fig 5 - Setup Admin Credentials]
> <img width="1019" height="768" alt="image" src="https://github.com/user-attachments/assets/8af7d9d3-89d6-4ac8-adc9-c0dbf03fe93b" /> [Fig 6 - Sign In]
> <img width="1024" height="766" alt="image" src="https://github.com/user-attachments/assets/ce883447-21f2-4a9d-8755-5da015c84157" /> [Fig 7 - Windows Server Dashboard]
> <img width="1024" height="767" alt="image" src="https://github.com/user-attachments/assets/7b8e7571-4ba0-4c91-b84f-58b0b19cebfa" /> [Fig 8 - Insert Guest Additions]
> <img width="1026" height="772" alt="image" src="https://github.com/user-attachments/assets/725321f8-d854-4f4e-8df1-8d91abac6edd" /> [Fig 9 - VBox Guest Additions]
> <img width="1028" height="772" alt="image" src="https://github.com/user-attachments/assets/f96946d0-2c08-49bb-ba88-beb1422f725a" /> [Fig 10 - Ran As Administrator]
> <img width="1025" height="770" alt="image" src="https://github.com/user-attachments/assets/d1b425b4-2b90-4b4f-b542-5c8b9c17e97d" /> [Fig 11 - Install Location]
> <img width="1022" height="767" alt="image" src="https://github.com/user-attachments/assets/64ef839c-16d5-4f64-bee1-4c4cd3f60a32" /> [Fig 12 - Choose Components]
> <img width="1020" height="769" alt="image" src="https://github.com/user-attachments/assets/1a93e4a7-8faa-4613-b801-5f3c24c4fffc" /> [Fig 13 - Installing G. Additions]
> <img width="1180" height="898" alt="image" src="https://github.com/user-attachments/assets/c7ae6c84-8787-40ae-b4d9-d117b202fa1b" /> [Fig 14 - Reboot]
> <img width="1108" height="630" alt="image" src="https://github.com/user-attachments/assets/14b68bcb-a1ce-41e8-bb6e-4f0ea9d33f93" /> [Fig 15 - Network Check]
> <img width="1391" height="836" alt="image" src="https://github.com/user-attachments/assets/23cedb54-b639-4cf8-aa74-10d48c9ce8a9" /> [Fig 16 - pfSense Dashboard]
> <img width="1385" height="731" alt="image" src="https://github.com/user-attachments/assets/61cb8d57-32d4-454e-b9ec-f2cba2733c3d" /> [Fig 17 - DHCP Leases]
> <img width="1388" height="1258" alt="image" src="https://github.com/user-attachments/assets/b1380b26-09a0-4cbe-9a7f-967d1f3d9be9" /> [Fig 18 - Static IP Configuration]
> <img width="1112" height="834" alt="image" src="https://github.com/user-attachments/assets/4cd094a2-b98f-412d-870f-e6bb1549a586" /> [Fig 19 - Command Network Check]






