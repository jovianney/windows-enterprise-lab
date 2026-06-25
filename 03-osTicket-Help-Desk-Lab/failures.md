# Failures & Troubleshooting — osTicket Help Desk Lab

## Incident 001 — Docker ARM64 Incompatibility
**Problem:** All available osTicket Docker images (osticket/osticket, campbellsoftwaresolutions/osticket) are amd64-only. Running them on ARM64 Ubuntu VM produced "exec format error" and containers crashed immediately after starting.

**What I tried:**
- osticket/osticket — exec format error
- platform: linux/amd64 override — still crashed, QEMU user-space emulation not supported in VirtualBox ARM64
- campbellsoftwaresolutions/osticket — also amd64 only, same result

**Fix:** Abandoned Docker entirely. Pivoted to native LAMP install (Apache2 + PHP + MySQL) directly on the Ubuntu VM. Native packages are ARM64 compatible.

**Lesson:** ARM64 limitations are real and recurring. Always verify Docker image architecture before planning a deployment on Apple Silicon.

---

## Incident 002 — Apache Serving Raw PHP Instead of Executing It
**Problem:** curl http://localhost/ returned raw PHP source code instead of rendered HTML. osTicket files were in place but Apache wasn't processing PHP.

**Cause:** libapache2-mod-php was not installed. Apache needs this module to hand PHP files to the PHP interpreter instead of serving them as plain text.

**Fix:** sudo apt install libapache2-mod-php -y && sudo systemctl restart apache2

**Lesson:** Apache and PHP are separate pieces. Installing PHP alone doesn't connect it to Apache — you need the mod_php bridge package.

---

## Incident 003 — Port Forwarding Guest Port Mismatch
**Problem:** http://localhost:8080 returned "site can't be reached" from Mac browser even though containers/Apache were running.

**Cause:** VirtualBox port forwarding rule had Guest Port set to 8080, but Apache listens on port 80 inside the VM.

**Fix:** Updated port forwarding rule — Host Port 8080 → Guest Port 80.

**Lesson:** Always match the Guest Port to whatever port the service actually listens on inside the VM, not what you want to access it on externally.
