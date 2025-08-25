# Breach Attack Simulation (BAS) – End-to-End Walkthrough

## 1) Executive Summary
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

**Agents/Logs**
- Wazuh agent on `dvwa-app`, `win-host` (and optionally on `kali-attacker` in local-only mode).
- Web/OS logs → Wazuh; OpenSearch Dashboards used for visualizations.

**Safety**
All testing occurs **inside an isolated lab network** under your control.

---

## 3) Objectives & Scope
- Emulate realistic adversary behaviors mapped to **MITRE ATT&CK**.
- Validate visibility and alerting in **Wazuh**.
- Produce artifacts (commands, logs, screenshots) for a public portfolio write-up.

Out of scope: persistence on Windows with AD abuse; destructive actions beyond controlled impact simulation.

---

## 4) Pre-Checks

### 4.1 Host Reachability
```bash
ping -c 2 10.10.10.10   # wazuh
ping -c 2 10.10.10.20   # dvwa
ping -c 2 10.10.10.30   # windows
