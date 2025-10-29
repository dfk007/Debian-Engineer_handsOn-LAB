## Debian Engineer Hands‑On Prep (Enterprise‑grade) — Turing / PradeepIT

### Topics to Cover (from `handsOn-Topic-Checklist.md`)
- Debian install and post‑install setup (essential tools, shell enhancements)
- APT package management (search/install/remove, deps, repos & GPG, pinning/holds, recovery)
- systemd (start/stop/enable, logs, boot analysis, custom services, dependencies, timers)
- Bash scripting & automation (functions/error handling, log parsing, deployment + rollback, cron)
- Networking (interfaces/routing/DNS), firewall (UFW/iptables), troubleshooting
- Security & hardening (SSH keys/hardening, Fail2ban, unattended security updates, permissions/ACLs)
- Containers (Docker engine, Dockerfile, docker‑compose)
- CI/CD (git workflows, GitHub Actions)
- Scenario drills (slow boot, disk cleanup, service failure recovery)

### Enterprise Workflow (how I’d run a Debian host in production)
1. Minimal, headless install
   - Netinst ISO; LVM; no desktop; only "SSH server" and "standard system utilities".
2. First login and baseline
   - Set FQDN/timezone; full update/upgrade; install essentials; enable bash‑completion.
3. SSH hardening (keep current session open)
   - Key‑only auth, `PermitRootLogin no`, `PasswordAuthentication no`, optional non‑22 port; test and restart.
4. Firewall policy (least privilege)
   - Default deny incoming, allow required ports (SSH/HTTP/HTTPS); enable and verify.
5. Automated security updates
   - `unattended‑upgrades` installed and configured; dry‑run verified.
6. Kernel/sysctl and integrity
   - Hardened sysctl (rp_filter, no forwarding unless needed); AIDE initialized and scheduled.
7. Auditing and monitoring
   - Journald forwarding to central SIEM; Fail2ban; install monitoring agent (e.g., Node Exporter/Zabbix/Nagios).
8. Automation posture (Infrastructure as Code)
   - Provision with Terraform/cloud‑init; configure with Ansible; manage via Git + CI/CD.

### 3‑Hour Sprint (before starting the assessment)
1) Base access and sudo
   - su -; apt‑get update && apt‑get install -y sudo; usermod -aG sudo $USER; newgrp sudo
2) System hygiene
   - sudo apt update && sudo apt upgrade -y
   - sudo apt install -y vim curl git htop net-tools dnsutils ufw fail2ban unattended-upgrades bash-completion
3) SSH hardening (validate with `sshd -t` before restart)
4) Firewall enablement (default deny; allow SSH port)
5) Smoke checks: systemd‑analyze, journalctl -p err, df -h, ss -tulpn

### Immediate next commands (if `sudo` is missing on the box)
```bash
su -
apt-get update && apt-get install -y sudo
usermod -aG sudo dfk
exit
newgrp sudo
sudo true && echo "sudo ready"
```

### Hands‑On Practice Flow (condensed)
1) Install & baseline: minimal install, hostname/timezone, essentials, `~/practice/{scripts,logs,backups}`
2) APT mastery: search/show/install/remove; dpkg -L/-s; fix‑broken; dpkg --configure -a; repos/keys; pin/hold
3) systemd: list/enable/disable, logs, boot analysis; write one simple service and one timer; `Restart=on-failure`
4) Scripting: disk check, log parser, deploy with rollback; schedule via cron or systemd timers
5) Networking & firewall: ip addr/route, static IP, DNS via systemd‑resolved; ping/ss/tcpdump; UFW + iptables basics
6) Security: SSH hardening; Fail2ban; unattended‑upgrades; permissions/ACLs; basic sysctl; AIDE
7) Containers & CI/CD: Docker install; small Nginx image + run; docker‑compose web+db; minimal GitHub Actions workflow
8) Scenario drills: slow boot triage, disk cleanup, service failure investigation and recovery

### How this maps to the role (PradeepIT / Turing JD)
- Manage Debian systems: enterprise workflow section (baseline, hardening, services, monitoring)
- APT expertise: APT mastery module, repo/keys, pinning, recovery
- Deployment workflows: deploy script with rollback; systemd services/timers; docker builds
- Troubleshooting: scenario drills; networking tools; systemd logs/boot analysis
- CI/CD and containers: Docker + compose; GitHub Actions
- Security posture: SSH hardening, firewall, unattended upgrades, AIDE, Fail2ban

### Evidence of readiness (what I’ll demo)
- Hardened SSH and active firewall; unattended‑upgrades configured
- Working custom systemd service and timer
- Scripts for disk checks, log parsing, and deploy with rollback
- Docker image built and running; compose stack up
- CI/CD workflow file present; git history with branches/merges
- Clean smoke checks: no critical logs, services healthy, ports intentional