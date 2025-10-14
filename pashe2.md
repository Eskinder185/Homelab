🔐 Security+ PseudoNAS — Secure File-Sharing and Monitoring Lab

![Linux](https://img.shields.io/badge/OS-Linux-blue?logo=linux)
![Proxmox](https://img.shields.io/badge/Hypervisor-Proxmox-red?logo=proxmox)
![Security+](https://img.shields.io/badge/Project-CompTIA%20Security%2B-yellow)
![Docker](https://img.shields.io/badge/Stack-Grafana%20%7C%20Loki%20%7C%20Prometheus-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 🧭 Overview

This project demonstrates **Confidentiality, Integrity, and Availability (CIA)** principles using a self-built NAS node on an **HP 15-bs0xx laptop** running Proxmox and Ubuntu Server.

It acts as a secure file-sharing and monitoring system, highlighting both **strengths** and **hardware limitations**.  
Built to align with **CompTIA Security+ (SY0-701)** objectives.

---

## ⚙️ Architecture

| Device | Role | Security Layer |
|---------|------|----------------|
| **HP 15-bs0xx (Proxmox Host)** | NAS Host + Monitor | Samba + AES Encryption + Fail2Ban + AIDE |
| **Main PC (Windows 11)** | Client | Access via SSH/SFTP and Tailscale |
| **External Drive (1 TB ext4)** | Backup | AES-256 Encrypted Backups |
| **Grafana Stack** | Visibility Layer | Loki + Prometheus + Grafana |

---

##  Phase-by-Phase Implementation

### **Phase 1 — Proxmox & Container Setup**

```bash
sudo pveversion
sudo pct list
sudo pct create 105 local:vztmpl/ubuntu-22.04-standard_22.04-1_amd64.tar.zst -rootfs local-lvm:10 -hostname nas-ct -net0 name=eth0,bridge=vmbr0,ip=dhcp
sudo pct start 105
sudo pct exec 105 -- bash
Phase 2 — Secure File Sharing (Samba + Encryption)
bash
Copy code
sudo apt update && sudo apt install -y samba openssl
sudo mkdir /secure_share
sudo useradd -M secureuser
sudo smbpasswd -a secureuser
Edit Samba Config

bash
Copy code
sudo nano /etc/samba/smb.conf
Add:

ini
Copy code
[SecureShare]
path = /secure_share
valid users = secureuser
read only = no
encrypt passwords = yes
smb encrypt = required
 In-transit encryption enabled (SMB3 AES-128-GCM).

Phase 3 — Encrypt Backups (Data at Rest)
bash
Copy code
tar -czf /tmp/backup.tar.gz /secure_share
openssl enc -aes-256-cbc -salt -pbkdf2 -iter 200000 \
  -in /tmp/backup.tar.gz \
  -out /mnt/external/backup.enc \
  -pass file:/root/.keys/backup.pass
rm /tmp/backup.tar.gz
 AES-256-CBC with PBKDF2 encryption protects backup data.

Phase 4 — Integrity Monitoring & Intrusion Detection
Fail2Ban (Detect Brute-Force)

bash
Copy code
sudo apt install -y fail2ban
sudo systemctl enable fail2ban --now
sudo cat /var/log/fail2ban.log
AIDE (File Integrity Monitoring)

bash
Copy code
sudo apt install -y aide
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
sudo aide --check
Phase 5 — Availability Testing
bash
Copy code
dd if=/dev/zero of=/secure_share/testfile bs=1M count=2048
htop
iostat -xm 2
Observe slowdown under heavy I/O to illustrate hardware resource limits.

Phase 6 — Security Monitoring Dashboard (Grafana + Loki + Prometheus)
bash
Copy code
sudo apt install -y docker.io docker-compose
mkdir -p ~/security-monitor && cd ~/security-monitor
nano docker-compose.yml
Paste:

yaml
Copy code
version: "3.8"
services:
  loki:
    image: grafana/loki:2.9.0
    ports: ["3100:3100"]
  promtail:
    image: grafana/promtail:2.9.0
    volumes:
      - /var/log:/var/log
  prometheus:
    image: prom/prometheus
    ports: ["9090:9090"]
  grafana:
    image: grafana/grafana
    ports: ["3000:3000"]
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
Start stack:

bash
Copy code
sudo docker compose up -d
Access Grafana:
👉 http://<TAILSCALE_IP>:3000
Login admin / admin → add Loki & Prometheus sources.

📊 Visualization Examples
Panel	Query	Description
🔐 Failed SSH Attempts	`{filename="/var/log/auth.log"}	= "Failed password"`
🚫 Banned IPs (Fail2Ban)	`{filename="/var/log/fail2ban.log"}	= "Ban"`
🧰 Samba Access Events	{filename="/var/log/samba/log.smbd"}	Monitor file-share access
💾 CPU Usage via Prometheus	rate(node_cpu_seconds_total[1m])	Check system load impact

🧾 Deliverables
File	Description
configs/smb.conf	Samba encryption config
scripts/backup_encrypt.sh	AES-256 backup script
evidence/htop_usage.png	Availability under load
evidence/fail2ban.log	Intrusion log evidence
report/risk_assessment.md	CIA risk summary

🧩 Risk Analysis
Category	Observation	Mitigation
Availability	System slowdown under encryption load	Upgrade RAM / offload to NAS hardware
Integrity	Backup corruption possible when RAM full	Validate with SHA256 hashes
Confidentiality	AES encryption causes CPU strain	Use hardware AES-NI or dedicated mini PC

📚 Security+ Domains Covered
Domain	Description
1.0 Threats, Vulnerabilities, and Mitigations	Detecting and responding to attacks (Fail2Ban, AIDE)
2.0 Security Operations	Log monitoring & incident response workflows
3.0 Architecture and Design	Encryption in transit & at rest
4.0 Governance, Risk, Compliance	Documenting CIA trade-offs and audit trails

