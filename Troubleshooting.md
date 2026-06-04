# Troubleshooting Log

A running log of issues encountered during lab setup and operations,
including root cause analysis and resolutions.

---

## Issue Index

| # | Issue | Component | Status |
|---|-------|-----------|--------|
| 001 | Domain user profile path not found | Active Directory / Windows 11 | ✅ Resolved |

---

## Issue 001 — Domain User Profile Path Not Found

**Date:** 06/03/2026
**Component:** Active Directory / Windows 11 VM
**Status:** ✅ Resolved

### Symptom
While logged in as Administrator on the Windows 11 VM, navigating to `C:\Users\pprice\Downloads` 
returned the error "The system cannot find the path specified." Additionally, attempting to log 
into the Windows 11 VM with the `pprice` domain account returned "The user name or password is 
incorrect."

### Environment
- **Domain Controller:** ECORP-DC (Windows Server 2025)
- **Endpoint:** Windows 11 VM (ECorp LAN — 10.0.1.x)
- **Domain:** ECORP.local
- **Affected Account:** pprice (domain user)

### Root Cause
Two compounding issues caused this:

1. **Profile folder did not exist** — User profile folders (`C:\Users\<username>`) are only 
created the first time a domain user logs into a machine. Since `pprice` had never logged into 
the Windows 11 VM, no profile folder existed on that endpoint. Running `dir C:\Users` on the DC 
confirmed only `Administrator` and `Public` were present.

2. **Password was incorrect or account needed a reset** — The `pprice` account credentials were 
not working at the Windows 11 login screen, preventing the first login that would have created 
the profile folder.

### Resolution
Reset the `pprice` account password directly from Active Directory Users and Computers on 
ECORP-DC, then logged into the Windows 11 VM using the new credentials. Windows automatically 
created the `C:\Users\pprice\` profile folder on first successful login.

### Steps Taken
1. On **ECORP-DC**, opened **Active Directory Users and Computers** via Start menu search
2. Located the `pprice` user account
3. Right-clicked `pprice` → **Reset Password**
4. Set a new password and unchecked **"User must change password at next logon"**
5. Verified the account was not locked out or disabled via **Properties → Account tab**
6. On the **Windows 11 VM** login screen, signed in as `ECORP\pprice` with the new password
7. Windows created the user profile folder at `C:\Users\pprice\` automatically on first login

### Screenshots
<img width="409" height="537" alt="image" src="https://github.com/user-attachments/assets/71e4c2fb-07ce-4735-a4fc-bff2096dbbeb" />

<img width="753" height="377" alt="image" src="https://github.com/user-attachments/assets/0182e228-3fcc-4c98-9f64-3e60b8a7c8c2" />

<img width="861" height="460" alt="image" src="https://github.com/user-attachments/assets/7c9549e1-c729-44d4-a4dc-a3bb410e16fa" />

<img width="863" height="431" alt="image" src="https://github.com/user-attachments/assets/9d85d779-84c8-49d3-a4bf-91c0929a4afe" />

<img width="858" height="434" alt="image" src="https://github.com/user-attachments/assets/ce82bac2-f2ff-4b27-8155-7bb7ee95b5d3" />

### Lessons Learned
- Domain user profile folders only exist on machines the user has physically logged into at 
least once — they are not pre-created by Active Directory.
- Always manage domain account passwords and properties from the Domain Controller via Active 
Directory Users and Computers, not from the endpoint.
- When a domain login fails, check the account status on the DC first (locked out, disabled, 
password expiration) before troubleshooting the endpoint.
- Using the `ECORP\pprice` format at the login screen explicitly tells Windows to authenticate 
against the domain rather than looking for a local account.



---
