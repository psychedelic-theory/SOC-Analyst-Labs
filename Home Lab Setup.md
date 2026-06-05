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
| [Install Windows Server](#wser) | Complete |
| [Install & Configure Active Directory](#active) | Complete |
| [Manage Users, Groups & Policies](#mugp) | Complete |
| [Domain Joining](#domainj) | Complete |
| [Install Sysmon](#sysmon) | Complete |
| [Install & Configure Splunk](#splunk) | In Progress |

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

#### Loggin in Requires Ctrl+Alt+Del Passthrough
Windows Server locks the screen and requires Ctrl+Alt+Del to unlock. In VirtualBox, pressing Ctrl+Alt+Del directly affects the host machine instead. To send it to the VM, go to **Input → Keyboard → Insert Ctrl-Alt-Del** from the VirtualBox menu bar.

#### Verify IP Is in the Correct Range
After first boot, open a Command Prompt and run `ipconfig` to confirm the server received an IP in the `10.0.1.x` range from pfSense's ECorp DHCP pool. If it shows something outside that range, check that the VM's network adapter is attached to the correct Internal Network (`LAN0`).

#### Assigning a Static IP via pfSense
Rather than configuring a static IP inside Windows, the static assignment is handled through pfSense's DHCP static mapping. Log into pfSense at `https://10.0.1.1`, navigate to **Status → DHCP Leases**, find the server's MAC address, and click the **+** icon to create a static mapping. Assign it `10.0.1.3`. After saving and applying changes, run `ipconfig /release` followed by `ipconfig /renew` in the server's Command Prompt to pick up the new address.

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

# <a name="active"></a>Install & Configure Active Directory

### Overview
This section covers promoting the Windows Server 2025 VM to a fully functioning Domain Controller for the ECorp network segment. The process involves renaming the server, installing the Active Directory Domain Services (AD DS) role, creating a new forest with the root domain `ECorp.local`, configuring Certificate Services (AD CS) to establish a Certification Authority, and setting a static IP directly on the server's network adapter. By the end of this section, the server operates as `ECorp-DC` — the authoritative domain controller for the lab environment — with a PKI infrastructure in place for future certificate-based authentication scenarios.

### Steps
**Install Active Directory**
- [X] Rename the server to `ECorp-DC` via System > About > Rename this PC and restart
- [X] After restart, open Server Manager > Manage > Add Roles and Features
- [X] Select Role-based or feature-based installation and click Next
- [X] Select the destination server from the server pool and click Next
- [X] Select **Active Directory Domain Services**, click Add Features, then click Next through Features and AD DS info screens
- [X] Check **Restart the destination server automatically if required**, confirm Yes, then click Install
- [X] After installation completes, click **Promote this server to a domain controller**
- [X] Select **Add a new forest** and enter `ECorp.local` as the root domain name, then click Next
- [X] Set a DSRM password on the Domain Controller Options screen and click Next
- [X] Click Next through DNS Options (ignore the delegation warning)
- [X] Wait for the NetBIOS domain name (`ECORP`) to auto-populate on the Additional Options screen, then click Next
- [X] Click Next through Paths and Review Options screens
- [X] After the prerequisites check passes, click Install — the server will restart automatically
- [X] Sign back in as `ECORP\Administrator`

**Configure Certificate Services**
- [X] Go to Manage > Add Roles and Features
- [X] Click Next through Before You Begin, Installation Type, and Server Selection screens
- [X] Select **Active Directory Certificate Services**, click Add Features, then click Next
- [X] Click Next through Features and AD CS info screens
- [X] Select **Certification Authority** as the role service, then click Next
- [X] Check **Restart the destination server automatically if required**, confirm Yes, then click Install
- [X] After installation, click **Configure Active Directory Certificate Services on the destination server**
- [X] Click Next on the Credentials screen (ECORP\Administrator is pre-filled)
- [X] Check **Certification Authority** as the role service to configure, then click Next
- [X] Select **Enterprise CA**, click Next
- [X] Select **Root CA**, click Next
- [X] Select **Create a new private key**, click Next
- [X] Keep the default SHA256 cryptography settings and click Next
- [X] Keep the default CA name and click Next
- [X] Keep the default validity period and click Next
- [X] Keep the default certificate database locations and click Next
- [X] Click **Configure** on the Confirmation screen
- [X] Confirm "Configuration succeeded" and reboot the server

**Set Network Connections**
- [X] Right-click the network icon in the system tray and open **Network & Internet settings**
- [X] Select **Ethernet > Edit IP Assignment**
- [X] Switch to **Manual**, enable IPv4, and enter: IP `10.0.1.3`, Subnet `255.255.255.0`, Gateway `10.0.1.1`, Preferred DNS `8.8.8.8`
- [X] Save and verify with `ipconfig /all` in Command Prompt

### Notes

#### Rename Before Prompting
Renaming the server before running the AD DS promotion wizard is important because the server name becomes embedded in the domain controller metadata, certificate common names, and DNS records during promotion. Changing the name after the fact requires additional steps to update AD objects and SPN registrations, and in some cases causes errors that are difficult to trace. `ECorp-DC` is descriptive and follows a naming convention that scales cleanly when additional domain controllers or member servers are added later.

#### The DNS Delegation Warning Is Expected
During the AD DS promotion wizard, the DNS Options screen displays a yellow warning stating that a delegation for the DNS server cannot be created because the authoritative parent zone cannot be found. This is normal in a lab environment where `ECorp.local` is a private domain with no upstream DNS authority. The warning can be safely ignored — DNS will function correctly within the internal network, and pfSense's DNS Resolver handles name resolution for the lab segments.

#### Why Enterprise CA Over Standalone
The setup type selection during AD CS configuration matters. An Enterprise CA integrates directly with Active Directory, allowing it to automatically issue certificates to domain members via Group Policy and store certificate templates in AD. A Standalone CA has no AD integration and requires all certificate requests to be handled manually. For a lab simulating an enterprise environment, Enterprise CA is the correct choice — it enables certificate auto-enrollment and supports scenarios like 802.1x, smart card auth, and LDAPS down the line.

#### Root CA vs. Subordinate CA
Selecting Root CA is appropriate here because there is no existing PKI hierarchy in the lab. A Subordinate CA would require a parent CA above it to sign its certificate — which doesn't exist in this environment. The Root CA is self-signed and sits at the top of the trust chain. In a production environment, the Root CA would typically be kept offline, with one or more Subordinate CAs handling day-to-day issuance, but for lab purposes a single Root CA is sufficient.

#### Static IP Set on the Adapter, Not via pfSense
Unlike the earlier static IP assignment for this server (which was handled through pfSense's DHCP static mapping), this step sets the IP directly on the server's network adapter via Windows network settings. Once a machine is a domain controller, it should have a static IP configured at the OS level rather than relying on DHCP — even a DHCP reservation — because AD, DNS, and Kerberos all expect the DC's IP to be stable and independently authoritative. The settings used are IP `10.0.1.3`, subnet `255.255.255.0`, gateway `10.0.1.1`, and preferred DNS `8.8.8.8`.

#### Verify with ipconfig /all
Running `ipconfig /all` after saving the static IP confirms the adapter picked up the correct address without needing a full reboot. The output should show `ECorp-DC` as the hostname, `Ecorp.local` as the primary DNS suffix, `10.0.1.3` as the IPv4 address, `10.0.1.1` as the default gateway, and `8.8.8.8` as the DNS server — all consistent with the ECorp LAN segment design.

### Screenshots
> <img width="1201" height="931" alt="image" src="https://github.com/user-attachments/assets/00da7686-e4d5-4eae-bfd4-92212a12ff65" /> [Fig 1 - Rename PC]
> <img width="1435" height="814" alt="image" src="https://github.com/user-attachments/assets/f4674c14-e040-4b39-b073-89ebecf51d2a" /> [Fig 2 - Add Roles & Features]
> <img width="1436" height="802" alt="image" src="https://github.com/user-attachments/assets/1e731543-e937-4aa9-be5f-ed1ccaf6b78c" /> [Fig 3 - Role-Based Installation]
> <img width="1432" height="808" alt="image" src="https://github.com/user-attachments/assets/b91d0475-aad1-4b0f-b784-fe81e16524b3" /> [Fig 4 - Destination Server]
> <img width="1433" height="800" alt="image" src="https://github.com/user-attachments/assets/6ed26ad6-843e-455b-9bb5-67abebf1a376" /> [Fig 5a - Server Role Selection]
> <img width="1438" height="805" alt="image" src="https://github.com/user-attachments/assets/055280d6-645d-47e1-8f4d-7bfcc63ccdb1" /> [Fig 5b - Active Directory Domain Service Role]
> <img width="1430" height="811" alt="image" src="https://github.com/user-attachments/assets/a1878ff7-da5c-4c1c-be2d-98c750ace4c8" /> [Fig 5c - Select Features List]
> <img width="1436" height="803" alt="image" src="https://github.com/user-attachments/assets/fea6ea87-77d5-4ab7-95af-2c200f894639" /> [Fig 6 - Active Directory Domain Services]
> <img width="1429" height="804" alt="image" src="https://github.com/user-attachments/assets/2ee7b272-c0ec-41a4-89f8-03a6bd6b3275" /> [Fig 7 - Confirm Installation & Restart]
> <img width="1435" height="799" alt="image" src="https://github.com/user-attachments/assets/ef36fbf6-9d86-4840-a4c5-bd3ac8ce75e3" /> [Fig 8 - Domain Controller/Installation Progress]
> <img width="1437" height="800" alt="image" src="https://github.com/user-attachments/assets/b53d561b-e471-4a8c-88e8-6c970907e183" /> [Fig 9 - Deployment Configuration 'New Forest']
> <img width="1432" height="804" alt="image" src="https://github.com/user-attachments/assets/d3e2de0d-b1d3-4cc6-8db0-73a21d5b8c7c" /> [Fig 10 - Directory Services Restore Mode (DSRM) Password Setup]
> <img width="1435" height="808" alt="image" src="https://github.com/user-attachments/assets/b008eb9c-b9b4-40d5-bdde-78280d0b4c02" /> [Fig 11 - DNS Options]
> <img width="1436" height="806" alt="image" src="https://github.com/user-attachments/assets/b97cbc7f-e11c-4c98-be22-a52526e72307" /> [Fig 12 - NetBIOS Domain Name]
> <img width="1435" height="800" alt="image" src="https://github.com/user-attachments/assets/5d76c7d8-76b7-457c-9a5d-5e0f051f979a" /> [Fig 13 - AD DS database, log files, and SYSVOL Paths]
> <img width="1432" height="802" alt="image" src="https://github.com/user-attachments/assets/881663aa-ee7a-41ff-a28f-536ebd66e885" /> [Fig 14 - Review Options]
> <img width="1428" height="802" alt="image" src="https://github.com/user-attachments/assets/1f3047b6-ed7e-48d8-a661-0234b0de28c8" /> [Fig 15 - Prerequisite Checks]
> <img width="1430" height="802" alt="image" src="https://github.com/user-attachments/assets/58c9e0b8-c789-4112-824d-b34c0a51c71a" /> [Fig 16 - System Restart]
> <img width="1433" height="807" alt="image" src="https://github.com/user-attachments/assets/fb53f459-1932-4c72-9855-0e009da41021" /> [Fig 17 - Role-Based Install #2]
> <img width="1438" height="808" alt="image" src="https://github.com/user-attachments/assets/a632aa5b-c1b4-4f02-8455-ba99e3c9692a" /> [Fig 18 - Server Dest. #2]
> <img width="1435" height="806" alt="image" src="https://github.com/user-attachments/assets/90b9d9ce-e133-4198-8d47-fb146f40ed95" /> [Fig 19 - Active Directory Certificate Services Role]
> <img width="1430" height="805" alt="image" src="https://github.com/user-attachments/assets/e0ce6709-45d1-4a96-a7ca-309f0a294c08" /> [Fig 20 - Add AD CSR features]
> <img width="1439" height="805" alt="image" src="https://github.com/user-attachments/assets/1521564a-194e-4702-88f8-b4c5da27e96d" /> [Fig 21 - Select Features]
> <img width="1432" height="802" alt="image" src="https://github.com/user-attachments/assets/ec0dec47-0dfc-4fd2-ad16-a793f5747f62" /> [Fig 22 - AD Certificate Services Info]
> <img width="1438" height="802" alt="image" src="https://github.com/user-attachments/assets/0a45df83-7919-472d-a15b-f2f2594d9eaf" /> [Fig 23 - Select Role Services]
> <img width="1437" height="810" alt="image" src="https://github.com/user-attachments/assets/badf4444-1d63-42c0-9892-c0c5e72afe60" /> [Fig 24 - Confirm Installation]
> <img width="1437" height="803" alt="image" src="https://github.com/user-attachments/assets/b602c719-c618-4a4a-bb01-261791f0ffea" /> [Fig 25 - Configure AD Cert. Services]
> <img width="1437" height="796" alt="image" src="https://github.com/user-attachments/assets/7d8adfdd-085f-4096-8381-2aa20f985bba" /> [Fig 26 - Credentials]
> <img width="1438" height="803" alt="image" src="https://github.com/user-attachments/assets/ece8a897-5620-46de-9221-c4ed0e638c01" /> [Fig 27 - Certificate Authority]
> <img width="1437" height="808" alt="image" src="https://github.com/user-attachments/assets/56d3485c-5692-4d33-adf9-6ffd3ea1786e" /> [Fig 28 - Enterprise CA]
> <img width="1432" height="806" alt="image" src="https://github.com/user-attachments/assets/6f99e016-f04c-4544-a6e1-3242840e996d" /> [Fig 29 - Root CA]
> <img width="1434" height="810" alt="image" src="https://github.com/user-attachments/assets/9ff26c51-65dd-4124-a313-30e32e077b2d" /> [Fig 30a - Private Key]
> <img width="1438" height="803" alt="image" src="https://github.com/user-attachments/assets/30bb7e60-b73b-47fd-8ce4-556cbc4caa9e" /> [Fig 30b - Cryptography for CA]
> <img width="1433" height="807" alt="image" src="https://github.com/user-attachments/assets/a8de097a-6759-490c-8581-e6ea6ac2accb" /> [Fig 30c - CA Name]
> <img width="1434" height="808" alt="image" src="https://github.com/user-attachments/assets/620aea59-8b55-44ac-83e1-df59a1afaf22" /> [Fig 30d - Validity Period]
> <img width="1435" height="801" alt="image" src="https://github.com/user-attachments/assets/4025acae-af5c-44bf-8a53-46b5bfb97cd0" /> [Fig 31 - CA Database]
> <img width="1440" height="805" alt="image" src="https://github.com/user-attachments/assets/6f9a7140-27fa-403a-a96b-17c99d059cf6" /> [Fig 32 - Confirmation]
> <img width="1430" height="808" alt="image" src="https://github.com/user-attachments/assets/1cbebe7b-57cf-4e0b-9c97-3293b6d96472" /> [Fig 33 - AD CS Configuration Success]
> <img width="804" height="631" alt="image" src="https://github.com/user-attachments/assets/27336273-1483-4b16-9bb8-ff8e3ca9a715" /> [Fig 34 - Network & Internet Settings]
> <img width="803" height="631" alt="image" src="https://github.com/user-attachments/assets/c40e741f-8ff9-4d67-9957-f3b598831e5a" /> [Fig 35 - Ethernet Settings]
> <img width="491" height="752" alt="image" src="https://github.com/user-attachments/assets/34732244-1b8a-45d6-81d4-2b52cdd035de" /> [Fig 36 - IPv4 IP Assignment]
> <img width="1109" height="595" alt="image" src="https://github.com/user-attachments/assets/fd483f97-f8e0-48ad-a790-4fec00c9b2f5" /> [Fig 37 - Network Config. Check]

# <a name="mugp"></a>Manage Users, Groups, & Policies

### Overview
This section covers organizing and populating the ECorp Active Directory environment with a realistic user and group structure. Using Active Directory Users and Computers, you will separate users and groups into dedicated Organizational Units, create domain administrator accounts, add standard user accounts, configure a service account with an intentionally weak password (for future attack simulation), set up a file share, register a Service Principal Name (SPN) for the SQL service account, and configure a domain-wide Group Policy Object (GPO) to disable Windows Defender across the domain. By the end of this section, ECorp.local reflects the kind of populated, misconfigured environment a SOC analyst would realistically encounter and investigate.

### Steps
- [X] Open Active Directory Users and Computers from Server Manager → Tools
- [X] Create a "Groups" Organizational Unit under ECorp.local
- [X] Move all default security groups from Users into the Groups OU
- [X] Copy the Administrator account to create a named domain admin user
- [X] Copy Administrator again to create an SQL Service account with a weak password stored in the Description field
- [X] Create additional standard user accounts
- [X] Set up a file share via File and Storage Services → Shares → New Share
- [X] Register an SPN for the SQL Service account using `setspn`
- [X] Verify the SPN registration with `setspn -T ECORP.local -Q */*`
- [X] Open Group Policy Management and create a new GPO linked to ECorp.local named "Disable Windows Defender"
- [X] Edit the GPO: Computer Configuration → Policies → Administrative Templates → Windows Components → Microsoft Defender Antivirus → Turn off Microsoft Defender Antivirus → Enabled
- [X] Set the GPO link to Enforced

### Notes
#### Separating Users and Groups Into OUs
By default, Active Directory places both user accounts and security groups inside the built-in Users container. Creating a dedicated "Groups" Organizational Unit and dragging all security groups into it keeps the directory readable and mirrors how mature enterprise environments structure their AD. When prompted with a warning about moving objects potentially affecting group policy application, confirm Yes — the groups being moved are security groups, not OUs with policy links, so there is no functional impact.

#### Copying Administrator to Create Named Admin Accounts
Rather than creating domain administrator accounts from scratch, copying the built-in Administrator account automatically inherits its group memberships — including Domain Admins. This is the fastest way to provision a named admin account with full domain privileges. Be sure to note the password, as there is no recovery prompt after creation.

#### The SQL Service Account Is Intentionally Misconfigured
The SQL Service account is created with a non-expiring password and has that password stored in plaintext in the account's Description field. This is a deliberate misconfiguration that mirrors a common real-world finding. It sets up a future attack scenario where an attacker with basic domain user access can read the Description field and obtain valid service account credentials. This is not sound practice, and the lab documentation calls this out explicitly.

#### SPN Registration and Kerberoasting Setup
Running `setspn -a ECorp/SQLService.ECORP.local:60111 ECORP\SQLService` registers the SQL Service account as a Kerberos service principal. Any domain user can then request a Kerberos service ticket for that SPN, which is encrypted with the service account's password hash — the foundation of a Kerberoasting attack. You can confirm the registration succeeded by running `setspn -T ECORP.local -Q */*` and locating the SQLService entry in the output.

#### Disabling Windows Defender via GPO
The "Disable Windows Defender" GPO is linked at the domain level, meaning it applies to all computers in ECorp.local. Setting the link to Enforced prevents any child OU-level policy from overriding it. This is another deliberate misconfiguration: in a real environment, disabling AV domain-wide is a significant security gap. In the lab, it prevents Defender from interfering with attack tools and payloads used in later exercises.

#### Apply Changes Is Not Optional
As noted in the pfSense configuration section, saving a policy change is not the same as applying it. After editing the GPO, confirm the settings are saved before closing the editor. On domain-joined endpoints, run `gpupdate /force` to push the policy immediately rather than waiting for the default refresh interval.

### Screenshots
> <img width="1435" height="856" alt="image" src="https://github.com/user-attachments/assets/a90e4fbd-92b8-448d-833b-69bd7530a73d" /> [Fig 1 - Active Directory Users and Computers]
> <img width="1439" height="858" alt="image" src="https://github.com/user-attachments/assets/b9bb6dfc-57f8-47f9-99cc-5b7e1ad7aa2c" /> [Fig 2 - Ecorp.local Users]
> <img width="1436" height="861" alt="image" src="https://github.com/user-attachments/assets/752a37d8-0640-4b54-b6b3-ec49ba1ee43c" /> [Fig 3 - New Organizational Unit]
> <img width="1435" height="858" alt="image" src="https://github.com/user-attachments/assets/3cf04de8-2706-4162-aeb2-373b2c6ab26a" /> [Fig 4 - Groups Unit]
> <img width="1434" height="854" alt="image" src="https://github.com/user-attachments/assets/987465ce-268d-44f6-a404-b77db70e543c" /> [Fig 5 - Move Users to Groups]
> <img width="1437" height="856" alt="image" src="https://github.com/user-attachments/assets/e2745a6d-e1db-4ab7-85d8-3d1db19892f2" /> [Fig 6a - Copy Administrator]
> <img width="1434" height="856" alt="image" src="https://github.com/user-attachments/assets/fb4d37b7-9092-4bfd-b96e-270147916afa" /> [Fig 6b - Create New Administrator]
> <img width="1432" height="851" alt="image" src="https://github.com/user-attachments/assets/680a79cc-a8c5-465e-822a-f4b3f8fb661f" /> [Fig 6c - Set Administrator Password]
> <img width="1438" height="860" alt="image" src="https://github.com/user-attachments/assets/1d0be3f0-b83f-4b84-bb44-758b65c10593" /> [Fig 6d - Finish Account]
> <img width="1435" height="855" alt="image" src="https://github.com/user-attachments/assets/f439350a-b92c-456b-baa5-7b24381969dc" /> [Fig 7a - SQL Administrator]
> <img width="1437" height="859" alt="image" src="https://github.com/user-attachments/assets/f64d02e6-dacb-4d61-be15-e6e191ade798" /> [Fig 7b - Password Creation]
> <img width="1435" height="865" alt="image" src="https://github.com/user-attachments/assets/741cbce5-311a-4339-9f5f-c8f40a2736f3" /> [Fig 7c - Finished SQL Admin Setup]
> <img width="1435" height="855" alt="image" src="https://github.com/user-attachments/assets/5d475acb-1dbe-4878-a1a1-db14c2db661c" /> [Fig 7d - SQLService Properties]
> <img width="1433" height="901" alt="image" src="https://github.com/user-attachments/assets/3a5a00bf-8306-44ef-9afc-d8eda2ebf776" /> [Fig 8a - New User]
> <img width="1440" height="859" alt="image" src="https://github.com/user-attachments/assets/2076e8d2-dae9-4a99-abc6-cc24de3674c4" /> [Fig 8b - Phillip Price User]
> <img width="1429" height="855" alt="image" src="https://github.com/user-attachments/assets/243d2988-3eeb-4eee-9f3c-2c2491ae952f" /> [Fig 8c - Price Password]
> <img width="1432" height="854" alt="image" src="https://github.com/user-attachments/assets/512380c7-8b58-4640-99c4-57eb4f265b42" /> [Fig 8d - Price Finished]
> <img width="1438" height="853" alt="image" src="https://github.com/user-attachments/assets/decb90d1-3a0f-4c01-9491-44a6a870e325" /> [Fig 9a - tcolby New User]
> <img width="1435" height="853" alt="image" src="https://github.com/user-attachments/assets/c31c7c71-e731-45b2-a2e1-664790a22169" /> [Fig 9b - tcolby password]
> <img width="1433" height="762" alt="image" src="https://github.com/user-attachments/assets/b1843338-529b-433a-a92e-1e1a86b93336" /> [Fig 9c - tcolby Finished]
> <img width="960" height="421" alt="image" src="https://github.com/user-attachments/assets/b53bf964-ef30-408d-aef9-a8c4c6fdea7a" /> [Fig 10 - File and Storage Devices]
> <img width="959" height="427" alt="image" src="https://github.com/user-attachments/assets/7ae99692-d53e-499a-a362-cb7b9fa4ba20" /> [Fig 11a - New Share]
> <img width="959" height="580" alt="image" src="https://github.com/user-attachments/assets/90e5249a-7e64-4d8c-ba55-d9a48d870e2f" /> [Fig 11b - File Share Profile]
> <img width="958" height="577" alt="image" src="https://github.com/user-attachments/assets/4cfe0a74-542d-4639-9d18-04e07a16e91f" /> [Fig 11c - Share Location]
> <img width="959" height="570" alt="image" src="https://github.com/user-attachments/assets/7163954b-b593-48cb-b719-50b354ed1b51" /> [Fig 11d - Share Name]
> <img width="960" height="577" alt="image" src="https://github.com/user-attachments/assets/bd86f7e6-2686-45a0-9362-b02b5dcb368a" /> [Fig 11e - Other Settings]
> <img width="960" height="579" alt="image" src="https://github.com/user-attachments/assets/a504cff0-dbec-4511-b402-3c803cf5c895" /> [Fig 11f - Permissions]
> <img width="959" height="571" alt="image" src="https://github.com/user-attachments/assets/558aa8f0-3d94-443a-8042-fee93a1ed171" /> [Fig 11g - Confirmation]
> <img width="958" height="574" alt="image" src="https://github.com/user-attachments/assets/8a30a064-2ea4-44cb-a6e2-1f132ec81c7c" /> [Fig 11h - Results]
> <img width="939" height="323" alt="image" src="https://github.com/user-attachments/assets/39e5c4f2-b323-4f03-872a-2882e8161282" /> [Fig 12a - Windows Powershell #1]
> <img width="941" height="626" alt="image" src="https://github.com/user-attachments/assets/9d4e3c42-5f0e-4125-b7cb-e9debf9c0a7e" /> [Fig 12b - Windows Powershell #2]
> <img width="956" height="806" alt="image" src="https://github.com/user-attachments/assets/d10301ac-98c8-41bc-8cf0-67a646b16586" /> [Fig 13a - Group Policy Management]
> <img width="749" height="530" alt="image" src="https://github.com/user-attachments/assets/6110bdeb-4a7d-4061-8c7a-bed94d04d356" /> [Fig 13b - Creating a GPO]
> <img width="752" height="529" alt="image" src="https://github.com/user-attachments/assets/a1c0781d-f63b-401a-8b53-50479ff334e2" /> [Fig 13c - Disable Windows Defender]
> <img width="757" height="528" alt="image" src="https://github.com/user-attachments/assets/030d7483-9fbc-4f3f-9375-f5bcb34931af" /> [Fig 13d - GPO Message]
> <img width="751" height="563" alt="image" src="https://github.com/user-attachments/assets/872e048a-bc42-4e56-a3a7-071e638f9265" /> [Fig 13e - Edit Policy]
> <img width="787" height="561" alt="image" src="https://github.com/user-attachments/assets/1257143e-89ac-41cb-ab0f-8d7621a7ba6c" /> [Fig 13f - Windows Components]
> <img width="827" height="562" alt="image" src="https://github.com/user-attachments/assets/093eb29f-b7a7-44ef-8b63-2c5571eed89d" /> [Fig 13g - Turn Off Microsoft Defender Antivirus]
> <img width="686" height="636" alt="image" src="https://github.com/user-attachments/assets/6f5a4bba-412b-4d45-a70b-8a1cb31b2857" /> [Fig 13h - Microsoft Defender Antivirus Settings]
> <img width="756" height="566" alt="image" src="https://github.com/user-attachments/assets/034c07a9-3ad7-47d7-9994-04addc3333cb" /> [Fig 13i - Set GPO to Enforced]

# <a name="domainj"></a>Domain Joining

### Overview
This section covers joining the Windows 11 VM to the ECorp.local domain created in the previous section. Before joining, the workstation's network adapter is reconfigured with a static IP and pointed to the Domain Controller's IP as its DNS server — a required step since Windows must be able to resolve the domain name through AD's DNS to complete the join. Once the static IP is set, the machine is joined through the Access Work or School settings, authenticated against the domain controller, and verified in Active Directory Users and Computers on the DC.

### Steps
- [X] Right-click the network icon in the system tray and open **Network & Internet settings**
- [X] Select **Ethernet**, then click **Edit** next to IP assignment
- [X] Change IP settings to **Manual** and configure IPv4: IP `10.0.1.2`, Subnet `255.255.255.0`, Gateway `10.0.1.1`, Preferred DNS `10.0.1.3`
- [X] Save the settings
- [X] Open the Start Menu search bar, type `Domain`, and select **Access work or school**
- [X] Click **Connect**
- [X] Select **Join this device to a local Active Directory domain**
- [X] Enter `ECorp.local` as the domain name and click **Next**
- [X] Authenticate with the domain Administrator credentials and click **OK**
- [X] Set the account type to **Administrator** and click **Next**
- [X] Click **Restart now** to reboot the machine and complete the join
- [X] On the Domain Controller, open **Server Manager → Tools → Active Directory Users and Computers**
- [X] Expand `ECorp.local`, select **Computers**, and verify the Windows 11 VM appears

### Notes
#### DNS Must Point to the Domain Controller
Before attempting to join the domain, the workstation's DNS server must be set to the Domain Controller's IP (`10.0.1.3`). Windows resolves `ECorp.local` through DNS during the join process — if DNS is pointing to an external server like `8.8.8.8`, the name lookup will fail and the domain join will error out. Setting the preferred DNS to the DC ensures that SRV records for Kerberos and LDAP are resolvable, which are required for the join to succeed.

#### Static IP on the Workstation
While the Windows 11 VM previously received its IP via DHCP (or through pfSense's static mapping), the domain join process is a good point to harden the network configuration. Assigning `10.0.1.2` statically at the OS level keeps the workstation's address stable and consistent with the lab's IP scheme, mirroring the same approach taken on the Domain Controller.

#### Join via Access Work or School, Not System Properties
Windows 11 removed the traditional **System Properties → Change** domain join path from easy access. The equivalent workflow is through **Settings → Accounts → Access work or school → Connect → Join this device to a local Active Directory domain**. The result is identical — the machine is joined at the OS level — but the navigation differs from older Windows versions.

#### Verify on the Domain Controller
After the workstation reboots, always confirm the join from the DC side by checking **Active Directory Users and Computers → ECorp.local → Computers**. The machine object appearing there confirms the join was recorded in AD and that the workstation is a recognized member of the domain. Checking only from the workstation side can miss cases where the join appeared to succeed locally but did not register properly in AD.

### Screenshots
> <img width="1010" height="753" alt="image" src="https://github.com/user-attachments/assets/21b1c72f-52c5-4ca8-a0af-2cb93928b792" /> [Fig 1 - Network & Internet Settings]
> <img width="1013" height="708" alt="image" src="https://github.com/user-attachments/assets/97182ee0-a127-4ad1-9f80-87c8c5866133" /> [Fig 2 - IP Assignment]
> <img width="1011" height="709" alt="image" src="https://github.com/user-attachments/assets/c76653dc-5e92-4017-bcc9-209c0eb6c7e7" /> [Fig 3 - Edit IP Settings]
> <img width="1011" height="752" alt="image" src="https://github.com/user-attachments/assets/c407ac23-2439-425f-b627-88e9e3cf4b52" /> [Fig 4 - Domain]
> <img width="1006" height="710" alt="image" src="https://github.com/user-attachments/assets/1c334de5-4e31-4be3-9f8a-7259b1317113" /> [Fig 5 - Connect Work Account]
> <img width="1014" height="709" alt="image" src="https://github.com/user-attachments/assets/9ce3d5f8-2754-4d2c-9e51-ab3e83919e7e" /> [Fig 6 - Join to Active Directory Domain]
> <img width="1010" height="710" alt="image" src="https://github.com/user-attachments/assets/6d95f719-9feb-4849-8146-5b18acae4293" /> [Fig 7a - Join The Domain]
> <img width="1009" height="709" alt="image" src="https://github.com/user-attachments/assets/3bc3de4f-1a23-4d44-8daa-7c05065edef3" /> [Fig 7b - Domain Account]
> <img width="1011" height="706" alt="image" src="https://github.com/user-attachments/assets/176e6e09-c14d-476e-8255-18d4e9d046e6" /> [Fig 7c - Add an account]
> <img width="1011" height="704" alt="image" src="https://github.com/user-attachments/assets/6194e40b-0037-41d2-ac32-6083311a3e61" /> [Fig 8 - Restart PC]
> <img width="959" height="898" alt="image" src="https://github.com/user-attachments/assets/25f66347-a009-4684-a16b-b6bc157c317a" /> [Fig 9 - Active Directory]
> <img width="956" height="894" alt="image" src="https://github.com/user-attachments/assets/e4b079b3-a37f-4972-a763-c1002659b02c" /> [Fig 10 - Computers]

# <a name="sysmon"></a>Install Sysmon

### Overview
Sysmon (System Monitor) is a Windows system service and device driver that remains resident across reboots to monitor and log system activity to the Windows Event Log. It provides granular telemetry on process creations, network connections, file creation time changes, and more — data that is critical for threat detection and log analysis in a SOC environment. This section covers installing Sysmon on the Windows 11 VM using the sysmon-modular configuration as a baseline, and verifying that events are being captured in Event Viewer.

### Steps
- [X] Download Sysmon from Microsoft Sysinternals
- [X] Download the sysmon-modular `sysmonconfig.xml` configuration file from GitHub
- [X] Copy the config XML text and save it as `sysmonconfig.xml` in Notepad
- [X] Unzip the Sysmon archive
- [X] Move `Sysmon64.exe` to the Downloads folder alongside `sysmonconfig.xml`
- [X] Open Command Prompt as Administrator and navigate to the Downloads folder
- [X] Run `sysmon64.exe -accepteula -i sysmonconfig.xml` to install
- [X] Open Event Viewer as Administrator using Domain Admin credentials
- [X] Navigate to Applications and Service Logs → Microsoft → Windows → Sysmon → Operational
- [X] Confirm Sysmon events are being logged

### Notes
#### Why sysmon-modular
Rather than writing a Sysmon configuration from scratch, the sysmon-modular project (maintained by olafhartong) provides a well-structured, community-vetted baseline. The default `sysmonconfig.xml` is the balanced configuration — most commonly used, covering a broad set of events without generating excessive noise. Any of these configurations should be treated as a starting point; tuning for the specific environment is strongly recommended over time.

#### Saving the Config File Correctly
When saving the configuration in Notepad, make sure to set the file type to **All Files** and name it exactly `sysmonconfig.xml`. Saving it as a `.txt` file with an `.xml` name is a common mistake — the extension must be correct for Sysmon to parse it properly.

#### Install Command Breakdown
The install command `sysmon64.exe -accepteula -i sysmonconfig.xml` does three things: `-accepteula` silently accepts the end user license agreement, `-i` specifies that this is an initial installation (as opposed to an update), and `sysmonconfig.xml` points to the configuration file. Both files must be in the same directory, or the full path to the config must be provided.

#### Verifying in Event Viewer
A successful installation can be confirmed by navigating to **Applications and Service Logs → Microsoft → Windows → Sysmon → Operational** in Event Viewer. If events are populating — process creations, registry queries, file operations — Sysmon is running and collecting telemetry correctly. Opening Event Viewer with **Run as Administrator** using Domain Admin credentials ensures full visibility into all administrative event logs.

### Screenshots
> <img width="1014" height="830" alt="image" src="https://github.com/user-attachments/assets/25c5d7cd-7969-4c58-aaeb-d010b2ba0e64" /> [Fig 1 - Sysmon Download]
> <img width="1012" height="782" alt="image" src="https://github.com/user-attachments/assets/b89b9512-8dc4-45c8-87c6-a9330a3bcc4b" /> [Fig 2 - Sysmon Repo]
> <img width="1010" height="785" alt="image" src="https://github.com/user-attachments/assets/d219da8a-15b1-4aaa-ad51-92092c9ad4a0" /> [Fig 3 - Sysmon Config]
> <img width="1015" height="787" alt="image" src="https://github.com/user-attachments/assets/6e4463c9-d21e-4ee2-8304-77c5624c9202" /> [Fig 4 - Configuration File]
> <img width="1011" height="781" alt="image" src="https://github.com/user-attachments/assets/0ec12b5e-9cb8-4cad-bbea-815227e929d5" /> [Fig 5 - File Extraction]
> <img width="1009" height="779" alt="image" src="https://github.com/user-attachments/assets/e5ac4fd0-07dd-464c-9eeb-940c2d163c4c" /> [Fig 6 - Sysmon64]
> <img width="457" height="604" alt="image" src="https://github.com/user-attachments/assets/4cc32281-b78d-4c71-a323-2b0e03956f8a" /> [Fig 7 - Sign In As Admin]
> <img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/2e96cc09-0f38-4dcb-8e32-73f72cea29f0" /> [Fig 8 - Run Program]
> <img width="770" height="721" alt="image" src="https://github.com/user-attachments/assets/1b21a2e3-c24f-43c4-8bcd-35021ccc1c66" /> [Fig 9 - Open Event Viewer]
> <img width="1089" height="726" alt="image" src="https://github.com/user-attachments/assets/2d9c997d-5ccc-4ca1-938c-c1407896c950" /> [Fig 10 - Sysmon > Operational]
> <img width="1084" height="718" alt="image" src="https://github.com/user-attachments/assets/cc42666c-51f7-4a78-81db-2453ed4c3ad0" /> [Fig 11 - Sysmon Logs]

# <a name="splunk"></a>Install & Configure Splunk

### Overview
This section covers installing Splunk Enterprise on the Windows 11 VM and configuring it to ingest endpoint telemetry from the ECorp network segment. Because the free Splunk license caps daily data ingestion at 500 MB, Splunk is installed on the Windows 11 VM only, and log collection is kept **disabled** until a test is actively being run — then disabled again immediately after to avoid hitting the limit. Once Splunk is running, two add-ons are installed from Splunkbase (the Sysmon Add-on and the Microsoft Windows Add-on) to give Splunk the field extractions needed to parse Windows and Sysmon event logs correctly. Data inputs are then configured via Remote Event Log Collections, and a quick verification search confirms that Sysmon telemetry is landing in the index.

### Steps
**Install Splunk**
- [ ] Sign up for a free Splunk account at [splunk.com/en_us/download/splunk-enterprise.html](https://www.splunk.com/en_us/download/splunk-enterprise.html) (a business email may be required)
- [ ] From the Windows 11 VM, select the **Windows** tab and download the 64-bit `.msi` installer
- [ ] Accept the Splunk terms of agreement on the download page
- [ ] Navigate to the Downloads folder and double-click the `.msi` file to launch the installer
- [ ] Accept the license agreement inside the installer and click **Next**
- [ ] Set an admin username and password when prompted, then complete the installation
- [ ] Launch Splunk Enterprise and, if it does not open automatically, navigate to `localhost:8000` in the browser
- [ ] Log in with the credentials created during installation
**Install Splunk Add-ons**
- [ ] Navigate to the Sysmon Add-on on Splunkbase: [splunkbase.splunk.com/app/5709](https://splunkbase.splunk.com/app/5709)
- [ ] Log in with your **Splunk account** (not your local Splunk instance credentials) and download the add-on
- [ ] In Splunk, go to **Apps → Manage Apps**, then click **Install App from File**
- [ ] Click **Upload**, navigate to the downloaded `.tgz` file, select it, and click **Upload**
- [ ] Confirm the success pop-up ("Splunk Add-on for Sysmon has been successfully installed")
- [ ] Return to Splunkbase, search for **"Windows"**, and select **Splunk Add-on for Microsoft Windows**
- [ ] Download and install it using the same **Install App from File** workflow
**Set Up Data Inputs**
- [ ] In Splunk, go to **Settings → Data inputs** (under the Data column)
- [ ] Select **Remote event log collections**
- [ ] Locate the `localhost` collection and click **Enable** to begin log ingestion
- [ ] Verify collection is working by running `ipconfig` from the Windows 11 command line
- [ ] Open the **Search & Reporting** app in Splunk and search for `"ipconfig"` — confirm one event is returned

### Notes

### Screenshots

















