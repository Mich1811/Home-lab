 Home Lab Setup (Pre-Attack)

This is the exact setup I used before starting the breach/attack simulations. Follow along step-by-step to replicate my environment.

---

## Topology & IP Plan

I isolated everything on a host-only network and added a NAT adapter for internet updates.

Host (laptop/desktop)
│
├─ VirtualBox Host-Only: vboxnet0 → 10.10.10.0/24 (no DHCP)
│
├─ Wazuh Manager (Ubuntu Server) .......... 10.10.10.10
├─ Ubuntu Web (DVWA on LAMP) .............. 10.10.10.20
├─ Windows 10 Workstation .................. 10.10.10.30
└─ Kali Linux Attacker ..................... 10.10.10.40

yaml
Copy
Edit

**Adapters on every VM**
- Adapter 1: **Host-Only** (vboxnet0)
- Adapter 2: **NAT** (for updates/package installs)

> Keep the host-only network private; never expose DVWA to the internet.

---

## Prerequisites

- VirtualBox (or VMware)
- ISOs: Ubuntu Server LTS, Kali, Windows 10
- Host-only network created: `vboxnet0` with `10.10.10.1/24`, DHCP **off**

---

## 1) Create VMs

I provisioned each VM with the following minimums and attached **Adapter 1: Host-Only** and **Adapter 2: NAT** before OS install.

| VM | vCPU | RAM | Disk | Notes |
|---|---:|---:|---:|---|
| Wazuh Manager (Ubuntu) | 2 | 4 GB | 40 GB | Server install |
| Ubuntu Web (DVWA) | 2 | 2 GB | 30 GB | LAMP stack |
| Windows 10 | 2 | 4 GB | 60 GB | RDP enabled |
| Kali Linux | 2 | 4 GB | 40 GB | Default tools |

Create an initial **snapshot** for each VM named `clean-install`.

---

## 2) Set Static IPs

### Linux (Ubuntu/Kali) — Netplan

I pinned the Host-Only adapter and left the NAT adapter on DHCP.

sudo nano /etc/netplan/01-netcfg.yaml

Windows 10 — GUI

Network & Internet → Change adapter options → Host-Only adapter → Properties → IPv4:

IP: 10.10.10.30

Mask: 255.255.255.0

Leave gateway/DNS empty on Host-Only (internet comes from NAT adapter)

IPs I used

Wazuh Manager: 10.10.10.10/24

Ubuntu Web (DVWA): 10.10.10.20/24

Windows 10: 10.10.10.30/24

Kali: 10.10.10.40/24

After doing this i checked for conectivity from kali:
ping -c 3 10.10.10.10   # Wazuh
ping -c 3 10.10.10.20   # DVWA
ping -c 3 10.10.10.30   # Windows


I installed Wazuh manager to monitor the endpoints.

# Repo & key
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg GPG-KEY-WAZUH
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
| sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
sudo apt -y install wazuh-manager
sudo systemctl enable --now wazuh-manager
sudo systemctl status wazuh-manager

Then check if UFW is enabled by running this:

sudo ufw allow 1514/udp
sudo ufw allow 1515/tcp
sudo ufw reload

Take a snapshot after this installation is complete.



