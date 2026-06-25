# 02 — Active Directory Home Lab

## Overview
Built a fully functional Windows Server 2025 Active Directory domain controller from scratch on Apple Silicon (M-series MacBook Air) using UTM/QEMU x64 emulation. Configured domain structure, user accounts, security groups, and Group Policy Objects mirroring a real enterprise environment.

## Environment
- **Host:** MacBook Air (Apple Silicon, ARM64)
- **Hypervisor:** UTM 4.7.5 (QEMU x64 emulation — required since ARM64 Windows Server builds are no longer hosted on Microsoft's servers)
- **Guest OS:** Windows Server 2025 Standard (Desktop Experience)
- **VM Specs:** 4GB RAM, 2 vCPUs, 64GB virtual disk
- **Domain:** jovilab.local
- **NetBIOS:** JOVILAB
- **DC Hostname:** DC01
- **Static IP:** 192.168.64.2

## What Was Built

### Step 1 — First Boot into Server Manager
![First login to Server Manager](screenshots/first-login-server-manager.png)

### Step 2 — AD DS and DNS Roles Confirmed
![Server Manager roles confirmed](screenshots/server-manager-roles-confirmed-AFTER.png)

### Step 3 — Domain Login Confirmed (JOVILAB\Administrator)
![JOVILAB domain login screen](screenshots/jovilab-domain-login-screen-AFTER.png)

### Step 4 — Creating Domain Users
![Creating intern01 user](screenshots/creating-intern01-user.png)

### Step 5 — Employees OU with 4 Users
![Employees OU with 4 users](screenshots/employees-ou-4-users.png)

### Step 6 — Security Groups Created
![Employees OU with users and groups](screenshots/employees-ou-users-and-groups.png)

### Step 7 — IT_Staff Group Members
![IT_Staff group members](screenshots/it-staff-group-members.png)

### Step 8 — Password Policy GPO
![GPO password length set to 12](screenshots/gpo-password-length-set-12.png)

![GPO password policy complete](screenshots/gpo-password-policy-complete.png)

### Step 9 — Account Lockout Policy
![GPO account lockout policy](screenshots/gpo-account-lockout-policy.png)

---

## User Accounts (all in Employees OU)
| Username | Full Name | Notes |
|----------|-----------|-------|
| jovi.d | Jovi De La Cruz | IT admin account |
| trainer01 | Trainer 01 | Standard user |
| manager01 | Manager 01 | Standard user |
| intern01 | Intern 01 | Standard user |

All accounts set to require password change on first logon.

## Security Groups
| Group | Type | Members |
|-------|------|---------|
| IT_Staff | Global Security | jovi.d, trainer01 |
| Managers | Global Security | manager01 |

## Group Policy — Default Domain Policy
**Password Policy:**
- Minimum password length: 12 characters
- Maximum password age: 90 days
- Password history: 10 passwords remembered
- Complexity requirements: Enabled

**Account Lockout Policy:**
- Lockout threshold: 5 invalid attempts
- Lockout duration: 10 minutes
- Reset counter after: 10 minutes

## Help Desk Simulations

### Password Reset
![Password reset right-click menu](screenshots/password-reset-right-click.png)

![Password reset dialog](screenshots/password-reset-simulation-BEFORE.png)

![Password reset confirmed](screenshots/password-reset-simulation-AFTER.png)

### Account Disable/Enable
![Account disabled via PowerShell](screenshots/account-lockout-simulation-BEFORE.png)

![Account is disabled checkbox](screenshots/account-disabled-BEFORE.png)

![Account re-enabled](screenshots/account-disabled-AFTER.png)

### Shared Folder with Group Permissions
![SharedDocs folder with IT_Staff permissions](screenshots/shared-folder-IT-staff-permissions-AFTER.png)

---

## Key Lessons
- ARM64 Windows Server builds are no longer hosted on Microsoft's servers — UTM/QEMU x64 emulation is the only viable path on Apple Silicon
- Server Manager "Roles: 0" is a known display bug in this Insider Preview build — clicking refresh resolves it
- DNS must point to itself on a domain controller, not the gateway
- Default Users container cannot have GPOs linked — always create custom OUs

## Resume Line
"Built Windows Active Directory domain with user management, GPOs, and security groups on Apple Silicon using UTM emulation"
