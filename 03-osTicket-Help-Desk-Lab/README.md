# 03 — osTicket Help Desk Lab

## Overview
Deployed osTicket v1.17.5 as a fully functional IT help desk ticketing system on a native LAMP stack (Linux, Apache, MySQL, PHP) inside an Ubuntu Server 26.04 LTS ARM64 VM running on VirtualBox. Simulated real-world IT support scenarios by creating and resolving mock tickets.

## Environment
- **Host:** MacBook Air (Apple Silicon, ARM64)
- **Hypervisor:** VirtualBox 7.2
- **Guest OS:** Ubuntu Server 26.04 LTS (ARM64)
- **VM Specs:** 2GB RAM, 2 vCPUs, 25GB virtual disk
- **Stack:** Apache2, PHP 8.5, MySQL 8.4, osTicket v1.17.5
- **Access:** SSH via VirtualBox port forwarding (Host 2222 → Guest 22)
- **Web Access:** http://localhost:8080 via port forwarding (Host 8080 → Guest 80)

## What Was Built

### Step 1 — Docker Installed (later abandoned)
![Docker install](screenshots/docker-install-AFTER.png)

### Step 2 — SSH Access Configured
![SSH connection from Mac terminal](screenshots/osticket-ssh-connection-AFTER.png)

### Step 3 — osTicket Installer (initial missing extensions)
![osTicket prerequisites with missing extensions](screenshots/osticket-installer-AFTER.png)

### Step 4 — All Prerequisites Met
![osTicket prerequisites all green](screenshots/osticket-prerequisites-AFTER.png)

### Step 5 — Installation Complete
![osTicket installation complete](screenshots/osticket-install-complete-AFTER.png)

### Step 6 — Post-Install Cleanup
![Post install cleanup commands](screenshots/osticket-post-install-cleanup.png)

### Step 7 — Login Page Live
![osTicket login page](screenshots/osticket-login-page-AFTER.png)

### Step 8 — Staff Control Panel
![osTicket dashboard](screenshots/osticket-dashboard-AFTER.png)

## Mock Ticket Simulations

### Ticket 1 — Password Reset
![Ticket 1 open](screenshots/ticket-01-password-reset-open.png)
![Ticket 1 resolved](screenshots/ticket-01-password-reset-resolved.png)

### Ticket 2 — Account Lockout
![Ticket 2 open](screenshots/ticket-02-account-lockout-open.png)
![Ticket 2 resolved](screenshots/ticket-02-account-lockout-resolved.png)

### Ticket 3 — Software Crash (Outlook)
![Ticket 3 open](screenshots/ticket-03-outlook-crash-open.png)
![Ticket 3 resolved](screenshots/ticket-03-outlook-crash-resolved.png)

## Key Lessons
- All osTicket Docker images are amd64-only — no ARM64 builds exist. Native install on Apache/PHP/MySQL is the correct approach on Apple Silicon
- PHP must be wired to Apache via libapache2-mod-php — without it Apache serves raw PHP code instead of executing it
- VirtualBox NAT port forwarding must map Host port → Guest port 80 (not 8080) for Apache to be reachable from the Mac browser
- SSH via port forwarding (Host 2222 → Guest 22) is essential for clipboard paste into headless VMs — eliminates manual typing of long commands

## Resume Line
"Deployed and managed osTicket help desk system on native LAMP stack with documented ticket resolution workflows"
