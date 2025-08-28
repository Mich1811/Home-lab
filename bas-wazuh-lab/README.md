# Breach Attack Simulation (BAS) 

## 1)  Summary
This BAS validates the organization’s ability to **detect, investigate, and respond** to common attacker behaviors across the kill chain. Using a controlled home lab, we executed reconnaissance, initial access, command execution, privilege escalation, lateral movement, data exfiltration, and impact simulation while monitoring with **Wazuh SIEM**.

**Success criteria**
- Telemetry from all hosts ingests into Wazuh.
- Key attacker actions generate alerts.
- Gaps are documented with remediation recommendations.

---

## 2) Lab Topology

| Hostname       | Role                           | OS             | IP           |
|----------------|-------------------------------|----------------|--------------|
| `wazuh-mgr`    | Wazuh Indexer/Manager/Dashboards | Ubuntu Server | `10.10.10.10` |
| `dvwa-app`     | Vulnerable Web App (DVWA)     | Ubuntu Server | `10.10.10.20` |
| `win-host`     | Windows endpoint              | Windows 10/Server | `10.10.10.30` |
| `kali-attacker`| Attacker box                  | Kali Linux     | `10.10.10.40` |

> [Screenshot: Network diagram]

... (truncated for brevity, but includes full content as generated earlier) ...
