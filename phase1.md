# LegacyLab — Phase 1 Report (What I Did)

**Date:** 2025-10-08  
**Host:** HP 15-bs0xx (2017), Intel i7, 12 GB RAM, 480 GB SSD

## TL;DR
- Installed **Proxmox VE** on the HP laptop.
- Installed and joined **Tailscale** to my tailnet.
- **Verified remote SSH** access to the HP from **outside** the home network (through Tailscale).
- Installed **Samba (SMB)** to act as a simple private file share.
- Kept the system lightweight to save RAM and disk for future phases.

---

## What I did (short bullets)
- **Base:** Booted the HP and confirmed **Proxmox VE** is installed and reachable on the LAN.
- **Secure remote access:**  
  - Installed **Tailscale** on the HP host.  
  - Logged in and **connected to tailnet**.  
  - From an external network, **SSH’d into the HP** using the Tailscale IP — confirmed I can manage it remotely.
- **File sharing:**  
  - Installed **Samba**.  
  - Created a basic private share for my own use (SMB, username/password).  
  - Mapped it from  client machine to confirm read/write works.
- **Resource discipline:**  
  - No heavy containers or VMs yet; focus on stability and remote control first.  
  - Left headroom for future monitoring and firewall/VLAN work.

---

## Proof I collected (or will add)
- [ ] Screenshot: **Tailscale admin** shows HP online  
- [ ] Screenshot: **SSH session** into HP via Tailscale IP (`whoami` + `hostname`)  
- [ ] Screenshot: **SMB share** visible from client (Explorer or `smbclient`)  
- [ ]  Screenshot: **Proxmox** page reachable on LAN


---

## Commands I used to verify (read-only checks)
```bash
# On the HP (or via SSH):
tailscale status
tailscale ip -4

# Confirm SSH is running
systemctl status ssh --no-pager

# Confirm Samba is running and config parses
systemctl status smbd --no-pager
testparm -s
Expected results

tailscale status shows this node connected.

You can SSH to the Tailscale IP from outside the home network.

smbd is active; testparm -s prints a clean config summary.

Limits / reality check (Phase 1)
No VLANs or firewall rules yet (that’s Phase 3).

No monitoring dashboards yet (that’s Phase 2).

Old hardware = limited RAM/CPU; I’m keeping services minimal to stay stable.

What’s next (Phase 2 preview)
Add Prometheus + Node Exporter for CPU/RAM/disk.

Add Loki + Promtail to collect auth and Samba logs.

Build a simple Grafana dashboard + basic alert.
