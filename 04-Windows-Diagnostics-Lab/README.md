# 04 — Windows Diagnostics Lab
**Platform:** DC01 — Windows Server 2025 VNext (Build 29602) via UTM 4.7.5 QEMU emulation  
**Domain:** jovilab.local  
**CompTIA A+ Core 2 Objectives:** 1.4, 2.2  

---

## Objective
Demonstrate proficiency with Windows built-in diagnostic and security tools used in real enterprise environments. All four labs run on the existing DC01 domain controller — no new VMs required.

---

## Lab 1 — Task Manager + Resource Monitor

### What I Did
Opened Task Manager and reviewed all four Performance tabs (CPU, Memory, Disk, Ethernet) on a live domain controller. Then opened Resource Monitor for deeper per-process analysis.

### Key Findings
- **CPU:** Spiked to 98% on boot due to QEMU x64-to-ARM64 emulation overhead. Settled to ~27% at idle. On real x64 hardware, Windows Server idle CPU should be under 15%.
- **Memory:** 2.1GB in use at idle out of 4GB allocated. Active Directory and DNS services are the primary consumers. 1.7GB cached for performance.
- **Disk:** Average response time of 271ms — high due to virtual disk emulation layers. Real SSD should be under 1ms; HDD under 20ms. Consistently high response times on real hardware indicate a failing drive or fragmentation.
- **Ethernet:** 0 Kbps at idle. Domain confirmed as jovilab.local, static IP 192.168.64.2. Periodic AD heartbeat spikes visible on graph.
- **Resource Monitor:** Identified per-process network connections — lsass.exe and dns.exe actively communicating with DC01.jovilab.local. Windows Defender (MsMpEng.exe) consuming 285MB RAM.

### Key Concepts
- Task Manager = quick overview, first tool opened during troubleshooting
- Resource Monitor = deep dive when Task Manager isn't enough
- High idle CPU on real hardware (>80%) = runaway process, malware, or hardware issue
- Low available memory (<200MB) consistently = RAM upgrade needed or memory leak

![Task Manager Performance CPU](screenshots/04-task-manager-performance-cpu.png)
![Resource Monitor Full Overview](screenshots/04-resource-monitor-full-overview.png)

---

## Lab 2 — Event Viewer

### What I Did
Reviewed all three primary Windows log types on DC01: Security, System, and Application. Filtered the Security log for specific event IDs and documented real events from the live domain controller.

### Key Findings

**Security Log — 14,170 events**
- Filtered for Event ID **4625 (Failed Logon)** — found 1 failed login attempt recorded on 6/22/2026 at 7:33 PM. Account: DC01$ on JOVILAB domain. This was triggered by a mistyped password during initial DC01 setup. In production, multiple 4625 events in rapid succession from the same account would indicate a brute force attack requiring immediate investigation.

**System Log — 8,190 events**
- Dominated by Event ID **7036 (Service Control Manager)** — services starting and stopping normally. No critical errors or warnings found. DC01 system health confirmed stable.

**Application Log — 890 events**
- Event ID **8198 (License Activation Failure)** — Windows unable to reach Microsoft activation servers. Error code: 0x87E10BC6. Expected behavior in an evaluation VM with no production license. In a real enterprise environment this would trigger a license compliance review and remediation via `slmgr /ato` or KMS server verification.

### Critical Event IDs to Know
| Event ID | Log | Meaning |
|----------|-----|---------|
| 4624 | Security | Successful logon |
| 4625 | Security | Failed logon — brute force indicator |
| 4634 | Security | Account logoff |
| 4672 | Security | Special/admin logon |
| 7036 | System | Service started or stopped |
| 7045 | System | New service installed — malware red flag |
| 41 | System | Unexpected shutdown |
| 6008 | System | Dirty shutdown logged on next boot |

### Connection to Level 5
The .evtx log files written by Windows are the same files Wazuh SIEM will ingest automatically at Level 5 — parsing these same event IDs across all machines and alerting on suspicious patterns in real time.

![Security Log 14170 Events](screenshots/04-event-viewer-security-log.png)
![Failed Logon Event 4625](screenshots/04-event-viewer-security-failed-logon-4625.png)
![Application Error 8198](screenshots/04-event-viewer-application-error-8198.png)

---

## Lab 3 — Windows Firewall Rules

### What I Did
Reviewed the default Windows Defender Firewall configuration on DC01, created a custom inbound block rule for TCP port 9999, verified it worked using PowerShell, then deleted the test rule.

### Key Findings
- **Three firewall profiles confirmed active:** Domain (primary on DC01), Private, Public
- **Default behavior:** Inbound connections blocked unless explicitly allowed. Outbound connections allowed unless explicitly blocked.
- **Active Directory firewall rules visible:** Ports 389 (LDAP), 445 (SMB), 636 (LDAPS), 3268, 3269 all open — required for domain controller functionality.
- **Custom rule created:** Block Port 9999 - LabTest — TCP inbound, all profiles, enabled
- **Verified with PowerShell:** `Test-NetConnection -ComputerName localhost -Port 9999` returned `TcpTestSucceeded : False`

### PowerShell Verification Output
ComputerName     : localhost

RemotePort       : 9999

PingSucceeded    : True

TcpTestSucceeded : False

### Key Concepts
- Always verify firewall rules after creation — don't assume they work
- Domain profile applies when joined to a domain network
- Public profile is most restrictive — used on untrusted networks
- Test-NetConnection is the go-to PowerShell command for port testing

![Firewall Overview](screenshots/04-firewall-overview.png)
![Custom Rule Created](screenshots/04-firewall-custom-rule-created.png)
![Port 9999 Blocked Test](screenshots/04-firewall-port-9999-blocked-test.png)

---

## Lab 4 — NTFS Permissions Deep Dive

### What I Did
Created a test folder (C:\NTFSTest), reviewed default inherited permissions, added domain user trainer01 with Read-only access, applied an explicit Deny on Write, then used Effective Access to verify the combined result.

### Key Findings
- **Default inherited permissions on new folder:** CREATOR OWNER, SYSTEM, JOVILAB\Administrators, JOVILAB\Users — all inherited automatically from C:\ drive
- **trainer01 added with Read permissions:** Read & Execute, List Folder Contents, Read — Allow. Write — explicitly Denied.
- **Effective Access result for trainer01:**
  - ✅ Can: Traverse folder, List data, Read attributes, Read permissions
  - ❌ Cannot: Create files, Write data, Create folders, Delete, Take ownership
  - Access limited by: File Permissions (explicit Deny rule)

### Critical Concept — Deny Overrides Allow
An explicit Deny always overrides Allow, regardless of group membership. Even if trainer01 were added to a group with Full Control, the explicit Write Deny on their account would still block write access. This is the most important NTFS permissions rule for the A+ exam.

### Key Concepts
- NTFS permissions are cumulative across groups — most permissive wins UNLESS there is an explicit Deny
- Inheritance flows from parent folder to child by default
- Effective Access tab shows the final calculated result after all rules are combined
- Principle of Least Privilege — grant only the minimum permissions required

![Default NTFS Permissions](screenshots/04-ntfs-permissions-default.png)
![trainer01 Read Permissions](screenshots/04-ntfs-trainer01-read-permissions.png)
![Effective Permissions trainer01](screenshots/04-ntfs-effective-permissions-trainer01.png)

---

## What I Learned
- Task Manager and Resource Monitor give layered visibility into system performance — start broad, drill down as needed
- Event Viewer is the first stop during incident investigation — Security log captures all authentication events permanently
- Windows Firewall rules must be verified after creation, not assumed to work
- NTFS Deny always overrides Allow — a single Deny beats 10 group-based Allows
- All four tools connect to real enterprise workflows: help desk triage, security investigations, access control, and compliance

---

## Files
04-Windows-Diagnostics-Lab/

├── README.md

├── failures.md

└── screenshots/

├── 04-task-manager-default-view.png

├── 04-task-manager-performance-cpu.png

├── 04-task-manager-performance-memory.png

├── 04-task-manager-performance-disk.png

├── 04-task-manager-performance-ethernet.png

├── 04-resource-monitor-overview.png

├── 04-resource-monitor-full-overview.png

├── 04-event-viewer-overview.png

├── 04-event-viewer-security-log.png

├── 04-event-viewer-security-failed-logon-4625.png

├── 04-event-viewer-system-log.png

├── 04-event-viewer-application-log.png

├── 04-event-viewer-application-error-8198.png

├── 04-firewall-overview.png

├── 04-firewall-inbound-rules.png

├── 04-firewall-custom-rule-created.png

├── 04-firewall-port-9999-blocked-test.png

├── 04-firewall-rule-deleted.png

├── 04-ntfs-permissions-default.png

├── 04-ntfs-trainer01-read-permissions.png

├── 04-ntfs-trainer01-deny-write.png

└── 04-ntfs-effective-permissions-trainer01.png

