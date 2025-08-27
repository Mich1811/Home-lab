##   Network Reconnaissance

**Objective**  
Conduct reconnaissance to identify live hosts, open ports, and running services within the target network.

**Methodology**  
- **Network range:** `10.10.10.0/24`  
- **Reconnaissance type:** Active scanning  

**Tools Used**  
- Nmap  

**Execution**  
```bash
nmap -sV -A 10.10.10.0/24

