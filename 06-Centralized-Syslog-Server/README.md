# Centralized Syslog Server (rsyslog + NXLog)

## Problem

Every machine in the lab — retropi, the Windows Server 2025 domain controller 
(DC01), and the Ubuntu VMs — was keeping its own local logs with zero 
visibility across machines. Troubleshooting meant SSHing or RDPing into each 
box individually. There was no single place to see what was happening across 
the whole environment, and no foundation in place for a future SIEM (Wazuh, 
Level 5).

## Goal

Stand up retropi as a central log collector. Get DC01 (Windows) shipping its 
Event Logs over the network in syslog format, have retropi receive and store 
them in their own file, and rotate that file automatically so it doesn't grow 
forever.

## Architecture
DC01 (Windows Server 2025)
└─ NXLog CE — reads Windows Event Log, converts to syslog, ships via UDP
│
▼
retropi (Raspberry Pi 5) — Tailscale IP 100.121.71.88
└─ rsyslog — listens on UDP 514, writes DC01 logs to /var/log/dc01.log
│
▼
logrotate — rotates dc01.log weekly, keeps 4 weeks, compresses old copies

## Solution

### 1. Install and configure NXLog CE on DC01

NXLog is the translator — Windows doesn't speak syslog natively, so NXLog 
reads the Windows Event Log and forwards it out as a syslog-formatted UDP 
packet.

Downloaded NXLog CE 2.8.1248 (official nxlog.co download flow was slow/gated 
on the emulated VM — grabbed the same file from a Software Informer mirror 
instead). Installed with defaults to `C:\Program Files (x86)\nxlog\`.

Confirmed the service was live in `services.msc`:

![NXLog service running](screenshots/nxlog-service-running.png)

### 2. First attempt failed — protocol mismatch

The sample config shipped with `om_tcp` as the output module. rsyslog's 
standard syslog reception expects UDP, so every connection attempt was 
rejected:

![NXLog TCP connection errors](screenshots/nxlog-log-before-udp-fix.png)

Fixed it by editing `nxlog.conf`'s Output block to use `om_udp` and point at 
retropi's Tailscale IP on port 514:

![NXLog conf with om_udp fix](screenshots/nxlog-conf-output-block.png)

After the fix, the connection errors stopped appearing entirely (UDP doesn't 
handshake, so no more errors — it just fires packets and moves on).

### 3. Enabled UDP reception on retropi

rsyslog ships with UDP input support built in but disabled by default. 
Uncommented the two lines in `/etc/rsyslog.conf`:
module(load="imudp")
input(type="imudp" port="514")

Opened the port:
```bash
sudo ufw allow 514/udp
sudo systemctl restart rsyslog
```

### 4. Found a second, unrelated bug — rsyslog wasn't writing to file at all

Even with the service running and the port confirmed listening 
(`ss -tulpn`), messages weren't landing in `/var/log/syslog`. Root cause: the 
default file-output rule in `/etc/rsyslog.conf` was commented out — likely 
disabled during the earlier OMV incident (Broken Lab 005). Uncommented the 
catch-all rule, restarted, confirmed with a manual `logger` test:

![rsyslog write rule fix confirmed](screenshots/rsyslog-write-rule-fix-confirmed.png)

### 5. Split DC01 into its own log file

Rather than mixing DC01's logs into the general `syslog` file alongside 
retropi's own noise (motion camera errors, etc.), added a filter rule:

`/etc/rsyslog.d/10-dc01.conf`:
if $fromhost-ip == '100.97.78.13' then /var/log/dc01.log
& stop

### 6. Configured logrotate for the new file

`/etc/logrotate.d/dc01`:
/var/log/dc01.log {
weekly
rotate 4
compress
missingok
notifempty
}

Verified with a dry run (`logrotate -d`) before trusting it to run for real.

## Proof

DC01 logs landing in their own separate file, confirmed by size and 
timestamp:

![DC01 log separation confirmed](screenshots/dc01-log-separation-confirmed.png)

Real Windows Event Log content flowing end to end from DC01 to retropi:

![DC01 log tail sample](screenshots/dc01-log-tail-sample.png)

## Result

Centralized syslog collection is live. DC01's Windows Event Logs ship over 
Tailscale via UDP, land in their own isolated file on retropi, and get 
rotated automatically. This becomes the log source for Wazuh SIEM at Level 5.