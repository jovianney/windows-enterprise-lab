# Failures & Troubleshooting — Windows Diagnostics Lab

## Issue 001 — Task Manager search took too long
**What happened:** Used Windows search bar to find Task Manager — took 2+ minutes to load results on the VM.  
**Fix:** Right-clicked taskbar to open Task Manager directly. Faster and more reliable on slow VMs.  
**Lesson:** On Windows Server VMs, always use direct methods — right-click, Run commands (Win+R), or taskbar shortcuts. Never rely on the search bar.

## Issue 002 — Resource Monitor couldn't find via Run
**What happened:** Tried opening Resource Monitor through search, VM was too slow to respond.  
**Fix:** Opened via Terminal using `resmon` command directly.  
**Lesson:** Memorize Run commands for all diagnostic tools — `resmon`, `eventvwr`, `wf.msc`, `perfmon`, `compmgmt.msc`. Faster than search every time.

## Issue 003 — License Activation Error 8198 in Application Log
**What happened:** Found Event ID 8198 errors in Application log — License Activation failure.  
**Root cause:** Windows Server 2025 VNext Preview (evaluation build) has no valid Microsoft license attached. VM cannot reach activation servers.  
**Fix:** Not applicable in lab environment — evaluation build by design.  
**Real world fix:** Run `slmgr /ato` to force activation attempt. If using KMS, verify KMS server is online and reachable. If retail license, re-enter key via `slmgr /ipk`.  
**Lesson:** Always distinguish between expected errors in lab environments vs actionable errors in production. Document both.