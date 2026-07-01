# Windows Command Cheat Sheet
*Real commands, plain English explanations — built while studying CompTIA A+ Core 2 and running a Windows Server 2025 domain lab.*

---

## Navigation

| Command | What it does |
|---|---|
| `dir` | Lists files and folders in current directory (Windows version of `ls`) |
| `cd foldername` | Move into a folder |
| `cd ..` | Go back one folder |
| `chdir` | Same as `cd` — shows or changes current directory |
| `mkdir foldername` | Create a new folder |
| `rmdir foldername` | Delete an empty folder |

---

## Disk Tools

| Command | What it does |
|---|---|
| `chkdsk` | Scans the disk and reports errors (read-only, doesn't fix) |
| `chkdsk /f` | Fixes filesystem errors it finds |
| `chkdsk /r` | Locates bad sectors and recovers readable data (includes /f) |
| `format D:` | Wipes and formats a drive (careful — destroys everything on it) |
| `diskpart` | Interactive disk partition manager — list, create, delete partitions |

---

## File Management

| Command | What it does |
|---|---|
| `copy file1 file2` | Copy a file |
| `copy /v file1 file2` | Copy and verify the file was written correctly |
| `copy /y file1 file2` | Copy and overwrite without asking for confirmation |
| `xcopy /s source dest` | Copy folders including subdirectories |
| `robocopy source dest` | Robust copy — survives network drops, keeps permissions |

---

## System Info & Repair

| Command | What it does |
|---|---|
| `whoami` | Shows which user account you're logged in as |
| `winver` | Shows the Windows version and build number |
| `sfc /scannow` | Scans and repairs corrupted Windows system files |
| `shutdown /r /t 0` | Restart the machine immediately |
| `tasklist` | See all running processes (Windows version of `ps aux`) |
| `taskkill /PID 1234` | Kill a process by its ID |

---

## Group Policy (Active Directory)

| Command | What it does |
|---|---|
| `gpupdate` | Refreshes Group Policy settings on the machine |
| `gpupdate /force` | Reapplies ALL policies, not just changed ones |
| `gpresult /r` | Shows which GPOs are actually applied to this user/machine |

*Used these on DC01 (jovilab.local) when applying GPOs across the domain.*

---

## Networking

| Command | What it does |
|---|---|
| `ipconfig` | Shows your IP address, subnet mask, and gateway |
| `ipconfig /all` | Full details — MAC address, DNS servers, DHCP lease info |
| `ipconfig /release` | Gives up your DHCP-assigned IP |
| `ipconfig /renew` | Requests a fresh IP from the DHCP server |
| `ipconfig /flushdns` | Clears the DNS cache (fixes stale DNS issues) |
| `ping 8.8.8.8` | Test connectivity to a host |
| `tracert google.com` | Shows every hop (router) between you and the destination |
| `pathping google.com` | Combines ping + tracert — shows packet loss per hop |
| `nslookup jovis.casa` | Asks DNS what IP a domain resolves to |

---

## Netstat

| Command | What it does |
|---|---|
| `netstat` | Shows active network connections |
| `netstat -a` | Shows ALL connections and listening ports |
| `netstat -b` | Shows which program owns each connection (needs admin) |
| `netstat -n` | Shows addresses as raw numbers instead of resolving names (faster) |

---

## Net Commands (Users & Shares)

| Command | What it does |
|---|---|
| `net view` | Lists computers and shared resources on the network |
| `net use Z: \\server\share` | Maps a network share to a drive letter |
| `net user` | Lists all user accounts on the machine |
| `net user intern01 /active:no` | Disables a user account |

*Used `net user` on DC01 for the account disable/enable simulation.*

---

## Key Concepts (not commands)

| Term | What it means |
|---|---|
| **TTL** | Time To Live — a counter in every packet that drops by 1 at each router hop. Hits 0 = packet dies. You see it in `ping` output (TTL=64 usually Linux, TTL=128 usually Windows) |
| **Packet loss** | Percentage of packets that never arrive. `ping` shows it as "% loss," `pathping` shows exactly which hop is dropping them |

---

## Linux ↔ Windows Quick Translation

| Linux | Windows | Job |
|---|---|---|
| `ls` | `dir` | List files |
| `pwd` | `cd` (no args) | Where am I |
| `cat` | `type` | Print file contents |
| `ps aux` | `tasklist` | List processes |
| `kill` | `taskkill` | Kill a process |
| `ip a` | `ipconfig` | Show IP info |
| `traceroute` | `tracert` | Trace network path |
| `df -h` | `chkdsk` / Disk Mgmt | Check disks |

---

*Built by Jovianney De La Cruz — windows-enterprise-lab*
