# Failures — Centralized Syslog Server

## Incident 1: TCP vs UDP mismatch on NXLog output

**What broke:** DC01's nxlog was configured with the `om_tcp` module 
(the sample config's default), throwing repeated 
`couldn't connect to tcp socket... actively refused` errors on every retry 
cycle.

**Root cause:** The default `nxlog.conf` sample ships with `om_tcp`, but 
rsyslog's standard syslog reception expects UDP. Never verified the sample 
matched the receiving service's expected protocol.

**Fix:** Changed the Output module from `om_tcp` to `om_udp` in `nxlog.conf`, 
restarted the service. Confirmed the fix by the *absence* of connection 
errors post-restart — UDP doesn't handshake, so a working UDP sender goes 
quiet instead of throwing errors when the receiver is unreachable.

**Lesson:** Don't assume a sample/default config matches your target 
service's protocol. Always check both ends agree before troubleshooting 
"why isn't this working."

## Incident 2: rsyslog receiving but silently not writing to file

**What broke:** rsyslog service showed `active (running)`, port 514 confirmed 
listening via `ss -tulpn`, and `journalctl` confirmed messages were actually 
being generated on the system — but `/var/log/syslog` stayed frozen at 0 
bytes, last modified three weeks prior.

**Root cause:** The default file-output rule in `/etc/rsyslog.conf` 
(`*.*;auth,authpriv.none -/var/log/syslog`) was commented out. Likely 
disabled during a prior config change tied to the OMV incident (Broken Lab 
005), and never restored afterward.

**Fix:** Uncommented the rule, restarted rsyslog, confirmed with a manual 
`logger` test message that it now correctly landed in `/var/log/syslog`.

**Lesson:** A running service is not a working service. `journalctl` showing 
the message existed system-wide (outside rsyslog entirely) was the key 
diagnostic step — it isolated the problem to rsyslog's file-writing rules 
specifically, ruling out the network path, the service itself, and the 
message generation all at once.

## Note: Windows Event ID 2050/2051 noise

DC01 generates frequent `INFO 2050` / `WARNING 2051` entries from 
`Microsoft-Windows-SystemDataArchiver` complaining about a missing "locale 
specific resource." This is a benign Windows quirk common on preview/eval 
builds (Server 2025 VNext) — not a real problem, just noisy background 
chatter. Confirmed logrotate will have real work to do given the volume.