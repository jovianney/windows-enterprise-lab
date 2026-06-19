# 13-Virtual-Firewall-Lab — Failures & Incident Log

---

## Incident 001 — Cloud-Init Boot Hang on Fresh Install
**Date:** June 2026
**Category:** Service Failure / Boot Process

**Problem:**
Fresh Ubuntu Server 26.04 install hung indefinitely at cloud-init-local.service during
every boot, including the installer's own SSH-info step.

**Symptoms:**
- Boot froze at "Starting cloud-init-local.service — Cloud-init: Local Stage (pre-network)..."
- No progress for 5+ minutes
- Reproduced across multiple fresh installs

**Fix:**
`sudo systemctl disable cloud-init cloud-init-local cloud-config cloud-final`

**Verification:**
sudo reboot completed in seconds, reached login prompt cleanly with no hang.

**Lesson:**
cloud-init is built for cloud providers (AWS/Azure) to auto-provision new instances — it
has no job on a manually-configured local VM and will hang trying to detect a cloud
datasource that doesn't exist. Disable it immediately on any local/offline VM install.

---

## Incident 002 — systemd-networkd-wait-online Hang
**Date:** June 2026
**Category:** Service Failure / Boot Process

**Problem:**
After fixing cloud-init, boot still occasionally hung at systemd-networkd-wait-online.service,
which waits for every detected interface to report "online" — including the isolated labnet
interface, which has no DHCP server and will never come online on its own.

**Symptoms:**
- Boot paused at "Job systemd-networkd-wait-online.service/start running"
- Eventually timed out and continued automatically after several minutes

**Fix:**
`sudo systemctl disable systemd-networkd-wait-online.service`

**Verification:**
Reboot reached login prompt immediately, no timeout delay.

**Lesson:**
Any VM with a deliberately-isolated network interface (no DHCP, no gateway) will trip
boot-time "wait for network" services. Disable them on purpose-built isolated network VMs.

---

## Incident 003 — NAT Does Not Equal Isolation
**Date:** June 2026
**Category:** Network Security / Firewall Logic
**Command Used:** `ping -c 4 192.168.12.1` (from isolated client VM)

**Problem:**
After configuring NAT (MASQUERADE) so the isolated labnet network could reach the internet,
assumed the isolation goal was already complete. Tested by pinging the home router directly
from the isolated client — it replied successfully, proving the "isolated" VM could actually
reach every device on the home network, not just the internet.

**Symptoms:**
- ping 8.8.8.8 (internet) succeeded — expected
- ping 192.168.12.1 (home router) also succeeded — NOT expected, defeats the isolation goal

**Fix:**
Added an explicit iptables FORWARD rule to drop traffic from the internal network destined
for the home subnet, before the default ACCEPT policy could pass it:
`sudo iptables -A FORWARD -i enp0s9 -d 192.168.12.0/24 -j DROP`
`sudo netfilter-persistent save`

**Verification:**
Re-ran both tests — 8.8.8.8 still 0% packet loss (internet intact), 192.168.12.1 now
100% packet loss (home network blocked).

**Lesson:**
NAT/MASQUERADE only handles address translation for outbound traffic — it does not
restrict destinations. A firewall has to explicitly define what's allowed AND what's
blocked. Getting internet access working is not the same as achieving network isolation;
both have to be tested and proven independently.

---
