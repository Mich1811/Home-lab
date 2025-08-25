# Home Lab Setup (Pre-Attack)

This is the exact setup I used before starting the breach/attack simulations. Follow along to replicate my environment.

---

## Topology & IP Plan

I isolated everything on a **host-only** network and added a **NAT** adapter for internet updates.

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

- VirtualBox
- ISOs: Ubuntu Server LTS, Kali, Windows 10
- Host-only network created: `vboxnet0` with `10.10.10.1/24`, DHCP **off**

---

## 1) Create VMs

I provisioned each VM with these minimums and attached **Host-Only** + **NAT** before OS install.

| VM | vCPU | RAM | Disk | Notes |
|---|---:|---:|---:|---|
| Wazuh Manager (Ubuntu) | 2 | 4 GB | 40 GB | Server install |
| Ubuntu Web (DVWA) | 2 | 2 GB | 30 GB | LAMP stack |
| Windows 10 | 2 | 4 GB | 60 GB | RDP enabled |
| Kali Linux | 2 | 4 GB | 40 GB | Default tools |

Create an initial snapshot for each VM named **`clean-install`**.

---

## 2) Networking & Static IPs

### Host-Only network
- Address space: `10.10.10.0/24`
- DHCP: **Disabled**

### Static IPs I used (Host-Only adapters)
- Wazuh Manager: `10.10.10.10/24`
- Ubuntu Web (DVWA): `10.10.10.20/24`
- Windows 10: `10.10.10.30/24`
- Kali: `10.10.10.40/24`

### Linux (Ubuntu/Kali)
- Configure the Host-Only interface with the static IP above using Netplan.
- Leave the NAT interface on DHCP (for internet access).
- Apply the network configuration and confirm the IPs.

### Windows 10
- Control Panel → Network & Internet → Change adapter options.
- Open the **Host-Only** adapter → Properties → IPv4 and set:
  - IP: your assigned `10.10.10.x`
  - Subnet Mask: `255.255.255.0`
  - Leave gateway/DNS empty on Host-Only (internet comes from NAT).

**Connectivity check**
- From Kali, ping `10.10.10.10`, `10.10.10.20`, and `10.10.10.30`.
- Ensure all hosts reply on the Host-Only network.

---

## 3) Base Hardening & Tools

### Linux VMs
- Update packages.
- Install essentials (curl, git, unzip, vim, net-tools, htop).
- Reboot if updates require it.

### Windows VM
- Enable RDP.
- Install a modern browser and PowerShell 7.
- Confirm Windows Defender and Firewall are enabled.

Snapshot all VMs: **`base-tools-ready`**.

---

## 4) Wazuh Manager (Ubuntu)

- Add the official Wazuh APT repository (follow Wazuh docs).
- Install **wazuh-manager**.
- Enable and start the service; confirm it’s running.
- If using UFW, allow ports **1514/udp** and **1515/tcp**.
- Verify the manager’s Host-Only IP is `10.10.10.10`.

Snapshot: **`wazuh-manager-installed`**.

---

## 5) Wazuh Agents

### Kali (Linux Agent)
- Install the Wazuh Linux agent package.
- During setup, set **Manager address** to `10.10.10.10`.
- Enable and start the agent service; confirm it’s active.

### Windows Agent
- Download and install the Wazuh Agent MSI.
- Set **Manager address** to `10.10.10.10` during installation.
- Start the agent service and verify it reports to the manager.

---

## 6) DVWA on Ubuntu Web

- Install **Apache**, **MariaDB**, and required **PHP** modules.
- Secure MariaDB (set root password).
- Create a **`dvwa`** database and a **`dvwa`** user with full privileges on that DB.
- Deploy DVWA from its GitHub repository into `/var/www/html/dvwa`.
- Copy the sample `config.inc.php` and update:
  - DB user: `dvwa`
  - DB password: the value you set
  - DB name: `dvwa`
- Set appropriate file ownership and permissions for the web server.
- In `php.ini` (Apache), enable `allow_url_fopen` and `allow_url_include`, then restart Apache.
- From Kali, browse to `http://10.10.10.20/dvwa`, click **Create / Reset Database**, and log in.
  - Defaults: `admin` / `password` → change after testing.

Snapshot: **`dvwa-ready`**.

---

## 7) Quality Checks

- **Reachability:** Each host responds to ping on the Host-Only network.
- **Web App:** `http://10.10.10.20/dvwa` loads from Kali.
- **Monitoring:** Wazuh Manager lists both the **Kali** and **Windows** agents as connected.
- **Logs:** Events from Kali and Windows are visible on the manager.

---





