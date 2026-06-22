# Failures & Troubleshooting — Active Directory Home Lab

---

## Incident 001 — ARM64 Windows Server ISO Not Available

**What happened:**
Attempted to download Windows Server ARM64 ISO directly from Microsoft's Insider Preview download page. Only x64 builds were available via direct download. Attempted to use UUP dump to build an ARM64 ISO from Microsoft's update servers — all ARM64 Server builds returned EMPTY_FILELIST error, meaning they had been removed from Microsoft's servers.

**What I tried:**
- Microsoft Insider Preview download page — no ARM64 Server option
- UUP dump build 26501.1000 (ge_prereleaseserver) arm64 — EMPTY_FILELIST
- Multiple older ARM64 Server builds on UUP dump — same error

**Root cause:**
Microsoft stopped hosting ARM64 Windows Server Insider Preview builds on their update servers. This is a known limitation — ARM64 Server builds were experimental and Microsoft quietly removed them.

**Fix:**
Pivoted to UTM (QEMU x64 emulation) instead of VirtualBox. Downloaded the standard x64 Windows Server 2025 Insider Preview ISO (which IS available) and ran it under full x64 emulation on Apple Silicon. Slower than native virtualization but fully functional for AD lab purposes.

**Lesson learned:**
ARM64 Windows Server is not a viable path on Apple Silicon right now. UTM/QEMU x64 emulation is the correct approach. Flag this for any future Windows Server lab work.

---

## Incident 002 — Server Manager "Roles: 0" Display Bug

**What happened:**
After successfully promoting DC01 to a domain controller, Server Manager Dashboard showed "Roles: 0" despite AD DS and DNS being fully installed and functional.

**What I tried:**
- Clicked refresh — sometimes resolved, sometimes didn't
- Verified AD DS was working via Tools → Active Directory Users and Computers — domain tree loaded correctly
- Verified DNS role active in left sidebar

**Root cause:**
Known display bug in Windows Server 2025 Insider Preview build. Server Manager's role count cache doesn't always refresh correctly after reboot in this build.

**Fix:**
Clicking the refresh icon in Server Manager resolves it temporarily. Confirmed roles are actually installed by checking the left sidebar (AD DS, DNS appear) and by successfully opening AD tools.

**Lesson learned:**
Don't trust the dashboard role count on Insider Preview builds. Verify functionality directly via the tools instead.

---

## Incident 003 — "Active Directory Domain Services Currently Unavailable" Error

**What happened:**
Opened Find Computers dialog from the desktop and received "The Active Directory Domain Services is currently unavailable" error.

**Root cause:**
The Find Computers shortcut on the desktop was trying to query AD before all services had fully initialized after boot. AD DS was actually running fine — the tool just launched too early.

**Fix:**
Closed the dialog and used Active Directory Users and Computers via Server Manager → Tools instead. No issues after that.

**Lesson learned:**
Some legacy Windows shortcuts and tools have race conditions with AD service startup. Always access AD tools through Server Manager → Tools for reliable results.
