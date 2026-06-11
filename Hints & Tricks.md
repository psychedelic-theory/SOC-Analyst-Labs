# Tips & Tricks

A running collection of hints, lessons learned, and best practices compiled from building and operating the ECorp home lab environment.

---

## Index

| # | Tip | Category |
|---|-----|----------|
| Host Hint 1 | Free Up Host RAM by Disabling SysMain | Host Machine |
| Host Hint 2 | Disable Windows Widgets | Host Machine / Windows VM |
| VM Hint 1 | Extend the Windows Evaluation License | Virtual Machines |
| VM Hint 2 | Snapshot Everything (Seriously) | Virtual Machines |
| VM Hint 3 | Understand Your VirtualBox Network Types | Virtual Machines |
| VM Hint 4 | pfSense Is Blocking You On Purpose (Implicit Deny) | Virtual Machines |
| VM Hint 5 | Always Test in This Order | Virtual Machines |
| VM Hint 6 | Start pfSense First, Always | Virtual Machines |
| VM Hint 7 | Static IPs Are Your Friend | Virtual Machines |
| VM Hint 8 | The Console in pfSense Is a Lifesaver | Virtual Machines |
| VM Hint 9 | Kali Problems Are Usually DNS, Not Kali | Virtual Machines |
| VM Hint 10 | Resource Allocation Matters | Virtual Machines |
| VM Hint 11 | If Something Is Weird, Reboot pfSense First | Virtual Machines |
| VM Hint 12 | You Are Building a Real Enterprise Network | Virtual Machines |

---

## Host Hint 1 — Free Up Host RAM by Disabling SysMain

### Overview
When running multiple VMs, host machine RAM is under heavy pressure. One hidden RAM consumer in Windows is the **SysMain** service (formerly Superfetch). SysMain tries to pre-load apps into memory for faster launches — great for normal users, terrible for a VirtualBox lab host.

### Steps
1. Press `Win + R` → type `services.msc`
2. Find **SysMain**
3. Right-click → **Stop**
4. Set **Startup type** to **Disabled**

### Notes
#### Why This Helps
SysMain aggressively consumes RAM for caching. Disabling it frees memory for pfSense, the Domain Controller, the Windows VM, and the Kali VM — your VMs get the RAM instead of Windows caching behavior.

#### Re-Enabling Later
This is a safe, reversible change. If you stop using the lab, simply set SysMain back to **Automatic**.

---

## Host Hint 2 — Disable Windows Widgets (Host and Windows VM)

### Overview
Windows 11 Widgets constantly run in the background, pulling web content, updating news and weather, consuming RAM and CPU, and generating background network traffic. This is useful for daily use but bad for a cybersecurity lab running multiple VMs.

### Steps
1. Right-click the taskbar → **Taskbar settings**
2. Turn **Widgets** → **Off**

### Notes
#### Do This On Both Machines
Apply this on your **host machine** and on your **Windows lab VM** after domain join. Widgets run even when you never open them — disabling them frees resources for VirtualBox, pfSense, the Domain Controller, and Kali. Less background noise means more RAM for your lab.

---

## VM Hint 1 — Extend the Windows Evaluation License

### Overview
Windows Server and some Windows images are evaluation copies. After a period of time, you will see "This copy of Windows is not genuine" warnings and random shutdowns every hour. This is not your lab breaking — it is the evaluation timer expiring. The fix takes 30 seconds.

### Steps
1. Open **Command Prompt as Administrator** inside the Windows VM
2. Run `slmgr /rearm`
3. Reboot the machine

### Notes
#### You Can Do This Multiple Times
The evaluation can be rearmed up to **6 times**, extending the lab life for many months. To check how much time is remaining before the next rearm is needed, run:
```
slmgr /dli
```

#### Why Students Get Tripped Up
Students often reinstall Windows, rebuild the domain, or conclude the lab is broken — when all that was needed was this one command. If your VM starts shutting down unexpectedly, run `slmgr /rearm` before doing anything else.

---

## VM Hint 2 — Snapshot Everything (Seriously)

### Overview
Snapshots in VirtualBox are saved states of a virtual machine at a specific point in time. When you take a snapshot, VirtualBox records the entire state of the VM — including its disk, memory, and settings — allowing you to return to that exact point later. This saves hours of unnecessary rebuild time.

### Steps
Take a snapshot at each of these milestones — before moving on to the next phase:
- Before you touch anything
- After pfSense is working
- After the Domain Controller is working
- After Windows joins the domain
- After Kali has internet

### Notes
#### Why This Is Non-Negotiable
This lab is meant to be broken. You are going to experiment, misconfigure rules, and test attacks. Snapshots turn disasters into 30-second fixes instead of multi-hour rebuilds.

#### What Snapshots Protect Against
- **Testing and Experimentation** — Try new software or configurations without risk. If something breaks, revert to the snapshot.
- **Safe Rollback** — Before making major changes like installing Guest Additions, changing network settings, or running potentially harmful code, a snapshot ensures you can undo the change.
- **Faster Recovery** — Instead of reinstalling or troubleshooting a broken VM, restore it to a known good state instantly.

---

## VM Hint 3 — Understand Your VirtualBox Network Types

### Overview
Most lab issues are not pfSense issues. They are VirtualBox network issues. Understanding the two relevant adapter types eliminates 90% of connectivity confusion.

### Notes
#### The Only Two You Need to Know
- **Bridged or NAT** → gives pfSense internet access
  - Bridged gets its IP from your home router
  - NAT gets its IP from VirtualBox
- **Internal Network** → connects all lab machines together

That is it. Every machine in the lab uses one or the other.

#### How to Diagnose Which Adapter Is Wrong
- If a machine can ping `8.8.8.8` but not other lab machines → wrong adapter
- If a machine can talk to others but not the internet → wrong adapter

90% of problems live here. Check the adapter before troubleshooting anything else.

---

## VM Hint 4 — pfSense Is Blocking You On Purpose (Implicit Deny)

### Overview
A common source of confusion: "I didn't block anything — why isn't it working?" You don't have to block anything. pfSense blocks everything unless you explicitly allow it. This is called implicit deny, and it is how real firewalls work.

### Notes
#### What This Explains
The implicit deny rule is why LAN works but WAN doesn't, why VLANs don't talk to each other by default, and why new interfaces are completely dead until rules are added. This is not a misconfiguration. This is intentional, correct firewall behavior.

#### The Fix
Review your firewall rules under **Firewall → Rules** for the relevant interface. If traffic is not passing, there is no rule allowing it. Add the rule, click **Apply Changes**, and retest.

---

## VM Hint 5 — Always Test in This Order

### Overview
When something does not work, do not guess. Test in this exact order to isolate the problem layer quickly.

### Steps
1. Can I ping the default gateway?
2. Can I ping another lab machine?
3. Can I ping `8.8.8.8`?
4. Can I resolve `google.com`?

### Notes
#### What Each Failure Tells You
- **Gateway fails** → IP address or adapter issue
- **Lab machine fails** → Internal network issue
- **8.8.8.8 fails** → pfSense rule or NAT issue
- **DNS fails** → DNS configuration issue

These four tests tell you exactly where the problem is without guessing.

---

## VM Hint 6 — Start pfSense First, Always

### Overview
The order in which you start your VMs matters. Starting the Windows workstation before pfSense means it will boot without a gateway, fail to reach the domain controller, and display errors that look like domain or network problems — but are actually just a sequencing issue.

### Steps
Start your VMs in this order every time:
1. pfSense
2. Domain Controller
3. Windows client
4. Kali (last)

### Notes
If you reverse this order, you will spend hours troubleshooting things that are not actually broken.

---

## VM Hint 7 — Static IPs Are Your Friend

### Overview
Do not rely on DHCP for your servers. Domain controllers, file servers, and SIEM nodes should all have static IP addresses — either assigned at the OS level or through pfSense DHCP static mappings. AD, DNS, and Kerberos all expect the DC's IP to be stable. A changing IP breaks domain resolution silently and is difficult to diagnose.

---

## VM Hint 8 — The Console in pfSense Is a Lifesaver

### Overview
If you lock yourself out of the pfSense web GUI, do not panic. The pfSense VM console menu is always accessible regardless of your firewall rules.

### Notes
#### What You Can Do From the Console
- Reset the web GUI password
- Reassign network interfaces
- Reset pfSense to factory defaults

Open the pfSense VM in VirtualBox and use the console menu directly. You are never truly locked out.

---

## VM Hint 9 — Kali Problems Are Usually DNS, Not Kali

### Overview
If Kali can ping `8.8.8.8` but `apt update` fails, that is not a Kali problem. That is DNS traffic through pfSense not being allowed. Students frequently reinstall Kali to fix this. Do not reinstall Kali.

### Steps
1. Confirm Kali has internet: `ping 8.8.8.8`
2. If ping succeeds but `apt update` fails, open pfSense and check your AttackLAN firewall rules
3. Verify that DNS traffic (UDP port 53) is permitted outbound from the AttackLAN interface

---

## VM Hint 10 — Resource Allocation Matters

### Overview
Starving VMs of RAM causes ghost problems — issues that look like networking or domain problems but are actually performance problems. Under-allocated VMs drop packets, time out on authentication, and behave erratically in ways that are nearly impossible to trace.

### Notes
#### Recommended Minimum RAM Allocation
| VM | Minimum RAM |
|----|-------------|
| pfSense | 2 GB |
| Domain Controller | 4 GB |
| Windows 11 | 4–6 GB |
| Kali Linux | 2 GB |

If things feel slow or buggy, check resource allocation before troubleshooting the network.

---

## VM Hint 11 — If Something Is Weird, Reboot pfSense First

### Overview
pfSense applies and maintains all firewall rules, DHCP leases, DNS resolution, and routing for the lab. If behavior is strange anywhere in the network, reboot pfSense before touching anything else. You would be surprised how often that is the fix.

---

## VM Hint 12 — You Are Building a Real Enterprise Network

### Overview
This lab behaves like a real company network — firewall in the middle, domain authentication, segmented machines, and a dedicated attacker machine. When something does not work, think like a network engineer, not like a home user.

### Notes
#### The Most Common Root Causes
If you remember nothing else, remember this. Most problems are caused by:
- Wrong adapter type in VirtualBox
- Missing firewall rule in pfSense
- DNS not being allowed through the firewall
- Forgetting to take a snapshot before making a change

This lab is not fragile. It is just realistic. And that is exactly what makes it so effective for learning cybersecurity.

---
