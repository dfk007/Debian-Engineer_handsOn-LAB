## Debian Engineer Hands‑On Prep (Turing / PradeepIT)

### What I Will Cover (from `handsOn-Topic-Checklist.md`)
- Debian install and post‑install setup (tools, shell QoL)
- APT package management (search/install/remove, repos & GPG, pinning/holds, recovery)
- systemd service management (start/stop/enable, logs, boot analysis, custom services, timers)
- Bash scripting & automation (functions/error handling, log parsing, deployments, cron)
- Networking (interfaces/routing/DNS), firewall with UFW/iptables, troubleshooting
- Security & hardening (SSH keys/hardening, Fail2ban, unattended upgrades, permissions/ACLs)
- Docker basics (engine, Dockerfile, compose)
- CI/CD (git workflows, GitHub Actions)
- Scenario drills (slow boot, disk cleanup, service failure recovery)
- Daily practice/test‑day checklist

### 3‑Hour Sprint (before starting the assessment)
1. Verify root and install sudo
   - su -
   - apt-get update && apt-get install -y sudo
   - usermod -aG sudo $USER; newgrp sudo
2. Essentials
   - sudo apt update && sudo apt upgrade -y
   - sudo apt install -y vim curl git htop net-tools dnsutils ufw fail2ban unattended-upgrades
3. SSH hardening (keep current session open)
   - Key auth, Disable root login and password auth; restart sshd
4. Firewall
   - sudo ufw default deny incoming; allow 22/tcp (or chosen port); enable
5. Smoke checks
   - systemd-analyze, journalctl -p err, df -h, ss -tulpn

### Hands‑On Flow To Practice (beginner → enterprise)
1. Install & baseline
   - Netinst, minimal, set hostname/timezone; create non‑root user
   - Post‑install: updates, essential packages, bash-completion; create `~/practice/{scripts,logs,backups}`
2. APT mastery
   - Search/show/install/remove; dpkg -L/-s; fix‑broken, dpkg --configure -a
   - Repos & keys; pin versions and holds; simulate and recover broken deps
3. Systemd
   - List services, enable/disable, analyze boot; create a simple custom service + timer
   - Read logs with journalctl; set Restart=on-failure
4. Scripting & automation
   - Write scripts: disk checks, log parser, deploy with rollback; wire via cron/systemd timers
5. Networking & firewall
   - ip addr/route; static IP; DNS via systemd-resolved; troubleshoot with ping/ss/tcpdump
   - UFW rules and iptables/nftables basics; persist rules
6. Security hardening
   - SSH hardening; Fail2ban; unattended‑upgrades; permissions & ACLs
   - Basic sysctl hardening; consider AIDE for integrity checks
7. Containers & CI/CD
   - Install Docker; build a small image (Nginx) and run
   - docker-compose for web+db; add a minimal GitHub Actions workflow
8. Scenario drills
   - Slow boot triage, disk space cleanup, service failure investigation and recovery

### How I Will Demonstrate Readiness
- Work through each module in `hands-on.md` under `~/practice` with saved commands/configs
- Keep a minimal log of what changed and why; revert skills (purge, re‑install, reconfigure)
- Produce a final checklist run: updates OK, ssh hardened, firewall active, services healthy, logs clean

### Immediate Next Command On The SSH Box (since `sudo` is missing)
```bash
su -
apt-get update && apt-get install -y sudo
usermod -aG sudo dfk
exit
newgrp sudo
sudo true && echo "sudo ready"
```


