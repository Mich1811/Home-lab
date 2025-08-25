# Home Lab Setup (Pre-Attack)

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

```bash
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

Connectivity check from Kali:

ping -c 3 10.10.10.10   # Wazuh
ping -c 3 10.10.10.20   # DVWA
ping -c 3 10.10.10.30   # Windows

3) Base Hardening & Tools

On Linux hosts I updated packages and installed essentials:

sudo apt update && sudo apt -y upgrade
sudo apt -y install curl git unzip vim net-tools htop


On Windows I enabled RDP, installed Chrome and PowerShell 7, and confirmed Defender + Firewall were on.

Snapshot: base-tools-ready.

4) Wazuh Manager (Ubuntu)

I installed Wazuh Manager to monitor endpoints.

# Repo & key
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg GPG-KEY-WAZUH
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
| sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
sudo apt -y install wazuh-manager
sudo systemctl enable --now wazuh-manager
sudo systemctl status wazuh-manager


If UFW is enabled:

sudo ufw allow 1514/udp
sudo ufw allow 1515/tcp
sudo ufw reload


Confirm manager IP on Host-Only is 10.10.10.10:

ip addr show


Snapshot: wazuh-manager-installed.

5) Wazuh Agents
Linux Agent (Kali)
curl -sO https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.8.0-1_amd64.deb
sudo WAZUH_MANAGER=10.10.10.10 dpkg -i ./wazuh-agent_4.8.0-1_amd64.deb
sudo systemctl enable --now wazuh-agent
sudo systemctl status wazuh-agent

Windows Agent

Download the Wazuh agent MSI.

During setup, set Manager address to 10.10.10.10.

Start the Wazuh agent service and verify it reports in.

6) DVWA on Ubuntu Web

I deployed a LAMP stack and DVWA.

# LAMP
sudo apt -y install apache2 mariadb-server php php-mysqli php-gd php-xml php-zip php-curl php-mbstring
sudo systemctl enable --now apache2 mariadb

# Secure MariaDB (interactive)
sudo mysql_secure_installation


Create DB and user:

sudo mysql -u root -p -e "CREATE DATABASE dvwa;"
sudo mysql -u root -p -e "CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'StrongP@ssw0rd!';"
sudo mysql -u root -p -e "GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost'; FLUSH PRIVILEGES;"


Deploy DVWA:

cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa
cd dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo sed -i "s/'user'.*'root'/'user' ] = 'dvwa'/" config.inc.php
sudo sed -i "s/'password'.*''/'password' ] = 'StrongP@ssw0rd!'/" config.inc.php
sudo sed -i "s/'database'.*'dvwa'/'database' ] = 'dvwa'/" config.inc.php


Permissions and PHP tweaks:

sudo chown -R www-data:www-data /var/www/html/dvwa
sudo find /var/www/html/dvwa -type d -exec chmod 755 {} \;
sudo find /var/www/html/dvwa -type f -exec chmod 644 {} \;

# Enable required PHP settings for DVWA
sudo sed -i 's/allow_url_fopen = .*/allow_url_fopen = On/' /etc/php/*/apache2/php.ini
sudo sed -i 's/allow_url_include = .*/allow_url_include = On/' /etc/php/*/apache2/php.ini
sudo systemctl restart apache2


Initialize DVWA in a browser (from Kali):

Open http://10.10.10.20/dvwa

Click Create / Reset Database

Login defaults: admin / password (change after testing)

Snapshot: dvwa-ready.

7) Quality Checks

From Kali:

# Reachability
for ip in 10.10.10.10 10.10.10.20 10.10.10.30; do ping -c 2 $ip; done

# DVWA page fetch
curl -I http://10.10.10.20/dvwa

# Wazuh agent service on Kali
systemctl is-active wazuh-agent


On the Wazuh Manager:

Confirm agents appear in the agent list.

Verify logs from Windows and Kali are ingested.
