#  LegacyLab — Phase 1 Report (System Setup & Remote Access)

**Date:** 2025-10-08  
**Host:** HP 15-bs0xx (2017) | Intel i7 | 12 GB RAM | 480 GB SSD  

---

##  TL;DR
- Installed **Proxmox VE 9** as a bare-metal hypervisor on the HP laptop.  
- Joined the host to my **Tailscale Tailnet** for secure remote connectivity.  
- Verified **remote SSH** access from outside my local network.  
- Installed **Samba (SMB)** for basic private file-sharing within the tailnet.  
- Kept the environment **lightweight and stable**, preserving memory and disk space for future monitoring and security phases.

---

##  What I Did (Step by Step)

### 1. Base Hypervisor Setup
- Booted into **Proxmox VE 9.0**.  
- Confirmed the web GUI (`https://<LAN_IP>:8006`) is reachable.  
- Verified Proxmox core services are active:
  ```bash
  sudo systemctl is-active pvedaemon pveproxy pvestatd pve-cluster
 All returned active.

2. Secure Remote Access via Tailscale
Installed and authenticated Tailscale:

bash
Copy code
sudo apt update && sudo apt install -y tailscale
sudo tailscale up --ssh
Confirmed the device joined my tailnet:

bash
Copy code
tailscale status
tailscale ip -4
From an external network, connected using SSH:

bash
Copy code
ssh analyst@100.xxx.xxx.xxx
 Successfully logged in — no router port forwarding needed.

3. Private File Sharing (Samba)
Installed and configured Samba:

bash
Copy code
sudo apt install -y samba
sudo mkdir -p /secure_share
sudo useradd -M secureuser
sudo smbpasswd -a secureuser
Edited /etc/samba/smb.conf:

ini
Copy code
[SecureShare]
path = /secure_share
valid users = secureuser
read only = no
encrypt passwords = yes
smb encrypt = required
Restarted and tested configuration:

bash
Copy code
sudo systemctl restart smbd
testparm -s
Mapped the share from a client:

php-template
Copy code
\\<TAILSCALE_IP>\SecureShare
 Read/write confirmed.

4. Resource Discipline
No containers or VMs yet — focus on stability and connectivity.

Left plenty of headroom for future monitoring stacks.

Verified idle load (htop) under 5% CPU and ~2 GB RAM usage.

📸 Proof Checklist
Evidence	Status
Screenshot: Tailscale Admin Console showing HP online	☐
Screenshot: SSH session via Tailscale IP	☐
Screenshot: SMB share visible from client	☐
Screenshot: Proxmox Web UI reachable on LAN	☐

 Verification Commands
bash
Copy code
# Tailscale connectivity
tailscale status
tailscale ip -4

# SSH service status
systemctl status ssh --no-pager

# Samba health
systemctl status smbd --no-pager
testparm -s
 Expected Results
tailscale status → node connected to tailnet.

SSH reachable externally (through Tailscale).

smbd active and configuration validated cleanly.

 Limits / Reality Check
No VLAN segmentation or firewall yet (coming in Phase 3).

No monitoring dashboards yet (coming in Phase 2).

Hardware constraints (2017 i7, 12 GB RAM) — keeping it efficient.

 Next (Phase 2 Preview)
Add Prometheus + Node Exporter to monitor CPU, memory, and disk.

Add Loki + Promtail to collect and centralize auth and Samba logs.

Create a Grafana Dashboard to visualize metrics and alerts in real time.
