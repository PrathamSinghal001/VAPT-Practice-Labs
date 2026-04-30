## 1. Nmap Full Port Scan with Service & OS Detection

### Command Used:

```bash
nmap 192.168.241.131 -p- -O -sV -oA metasploitable_scanning_reports
```

### Objective:

- Identify all open ports on the target system
- Detect running services and their versions
- Identify the operating system of the target
- Save output in multiple formats for further analysis

### Output Files Generated:

- metasploitable_scanning_reports.nmap → Normal output
- metasploitable_scanning_reports.xml → XML format (used for conversion)
- metasploitable_scanning_reports.gnmap → Grepable format

### File Location (Kali Linux):

```
/home/kali/metasploitable_scanning_reports.*
```

### Analysis:

The scan helps in identifying exposed services and potential attack vectors. Service version detection is useful for finding known vulnerabilities.

---
## 2 Convert XML format file into HTML format

### Command Used:

```bash
xsltproc -o metasploitable_scanning_reports.html /usr/share/nmap/nmap.xsl metasploitable_scanning_reports.xml
```

### Objective:

- Convert XML scan results into a human-readable HTML report

### Output File:

- metasploitable_scanning_reports.html

### File Location:

```
/home/kali/metasploitable_scanning_reports.html
```

### Analysis:
The HTML report provides a structured and easy-to-read visualization of scan results, useful for reporting and documentation.

---

## 3. SMB Enumeration using enum4linux  
  
### Command Used:  

```bash  
enum4linux -A 192.168.241.131
```

### Objective:

- Enumerate SMB service
- Identify users, shares, password policy, and system information

### Key Findings:

#### 1. Workgroup Information

- Target system is part of WORKGROUP (not a domain)

#### 2. Null Session Enabled

- SMB allows anonymous login (no username/password required)
- This is a critical vulnerability

#### 3. OS Information

- Samba version: 3.0.20-Debian
- Outdated version with known vulnerabilities

#### 4. User Enumeration

- Multiple users identified including:
    - root
    - msfadmin
    - mysql
    - postgres
    - www-data

#### 5. Share Enumeration

- Accessible share found:
    - tmp → readable

#### 6. Password Policy Weakness

- Minimum password length: 0
- Password complexity: Disabled

### Analysis:

The system is highly vulnerable due to weak security configurations. Anonymous SMB access and weak password policies significantly increase the attack surface.

### Conclusion:

SMB enumeration revealed multiple misconfigurations that can be exploited in further penetration testing phases such as brute force attacks, SMB exploitation, and privilege escalation.

---

## 4. Advanced Nmap Scan Results  
  
### Command Used:  

```bash  
nmap -A -T4 192.168.241.131
```

### Key Findings:

#### 1. FTP Service

- vsftpd 2.3.4 detected
- Anonymous login allowed
- Vulnerable to backdoor exploit

#### 2. SSH Service

- OpenSSH 4.7 detected
- Potential for brute force attack

#### 3. Telnet Service

- Unencrypted remote access
- High security risk

#### 4. SMB Service

- Samba 3.0.20 detected
- Known vulnerabilities and weak security configuration

#### 5. Web Server

- Apache 2.2.8 running
- Potential web vulnerabilities

#### 6. Root Shell Exposure

- Port 1524 exposes a root shell
- Critical vulnerability

#### 7. Database Services

- MySQL and PostgreSQL exposed

#### 8. Tomcat Server

- Apache Tomcat running on port 8180

#### 9. IRC Service

- UnrealIRCd running (known vulnerable version)

### Analysis:

The system exposes multiple vulnerable services, making it highly susceptible to exploitation.

### Conclusion:

The target system is critically vulnerable and can be easily compromised using multiple attack vectors.

---

## 5. SMB Share Access  
  
### Command Used:  
```bash  
smbclient //192.168.241.131/tmp -U msfadmin
```
### Objective:

- Access shared directory using valid credentials
- Identify accessible files and directories

### Findings:

- Successfully accessed `/tmp` share
- Multiple directories found including:
    - orbit-msfadmin
    - gconfd-msfadmin

### Analysis:

The ability to access SMB shares using valid credentials indicates weak access control. This can be leveraged to explore sensitive files or upload malicious payloads.

### Conclusion:

SMB access provides a foothold into the system, which can be used for further exploitation such as credential harvesting or remote code execution.

---

## 6. FTP Access

### Command Used:

```bash
ftp 192.168.241.131
```

### FTP Findings Update:

- Anonymous login successful
- Directory listing available but empty
- No sensitive files found
- FTP appears to have limited access

### Analysis:

Although anonymous access is allowed, no sensitive data was found. However, the FTP version (vsftpd 2.3.4) is known to be vulnerable and can be exploited.

### Conclusion:

FTP service is vulnerable but not useful for direct data extraction. It can be used for exploitation instead.

---

## 7. Vulnerability Assessment using Nmap NSE  
  
### Command Used:  
```bash  
nmap --script=vuln -T4 192.168.241.131
```
### Objective:

- Identify known vulnerabilities using Nmap scripting engine
- Validate exploitable services on the target system

## Key Findings:

### 1. FTP Service (Critical)

- Service: vsftpd 2.3.4
- Vulnerability: Backdoor (CVE-2011-2523)
- Impact: Remote attacker can gain root access without authentication

### 2. Web Server (High Risk)

- SQL Injection vulnerabilities detected in multiple endpoints
- Slowloris DoS vulnerability present
- HTTP TRACE method enabled

### 3. SSL/TLS Vulnerabilities

- Logjam attack (CVE-2015-4000)
- POODLE attack (CVE-2014-3566)
- Weak Diffie-Hellman parameters
- Impact: Susceptible to Man-in-the-Middle attacks

### 4. RMI Service (High Risk)

- Vulnerable to remote code execution
- Allows loading of remote classes

### 5. IRC Service (Critical)

- UnrealIRCd backdoor detected
- Allows remote command execution

### 6. Tomcat Server Issues

- Multiple admin panels exposed
- Session security misconfiguration (HttpOnly flag not set)

## Analysis:

The target system contains multiple critical vulnerabilities including backdoors, weak encryption, and remote code execution flaws. Several services allow unauthorized access or can be exploited remotely.

## Conclusion:

The system is highly vulnerable and can be fully compromised using multiple attack vectors. Immediate exploitation is possible via FTP, IRC, and RMI services.

---

## 8. Vulnerability Classification

### Critical Vulnerabilities (3)
- FTP Backdoor (vsftpd 2.3.4)
- IRC Backdoor (UnrealIRCd)
- Bindshell (Metasploitable root shell)

Impact:
- Direct remote root access
- Full system compromise
- Allows direct remote access to the system  
- No authentication required  
- Possible root-level access

### High Vulnerabilities (4)
- RMI Remote Code Execution
- SQL Injection (Web Application)
- Logjam SSL Vulnerability
- POODLE SSL Vulnerability

Impact:
- Remote code execution
- Data breach
- Encryption compromise

### Medium Vulnerabilities (5)
- Slowloris DoS
- CSRF vulnerabilities
- Exposed Admin Panels
- Weak Diffie-Hellman encryption
- Missing HttpOnly cookie flag

Impact:
- Service disruption
- Session hijacking
- Misconfiguration exploitation

### Low Vulnerabilities (2)
- HTTP TRACE enabled
- Information disclosure (directory listing)

Impact:
- Minor information leakage

### Conclusion:
The system contains multiple critical and high-risk vulnerabilities, making it highly exploitable. Immediate remediation is required.

---

## 9. Access vs Exploitation:

- FTP and SMB allowed access due to weak configuration
- No exploitation required for initial access

- Critical vulnerabilities such as FTP backdoor and IRC backdoor allow exploitation leading to remote root access

Conclusion:
Some services allow direct access due to misconfiguration, while others require exploitation to gain control.

---

## 10. SMB Enumeration and Vulnerability Analysis  
  
### Command Used:  
```bash  
nmap -p139,445 --script smb-enum*,smb-vuln* -sV 192.168.241.131
```
### Objective:

- Enumerate SMB users and shares
- Identify SMB-related vulnerabilities

### 1. User Enumeration

- Multiple users identified:
    - msfadmin
    - root
    - mysql
    - postgres
    - www-data
- Impact: Enables brute-force and credential-based attacks

### 2. Anonymous Share Access (Critical)

- Share: tmp
- Permissions: READ/WRITE without authentication
- Impact:
	- Unauthorized file access
	- File upload possible
	- Potential remote code execution

### 3. IPC Share Misconfiguration

- Anonymous access allowed
- Impact:
	- Information disclosure
	- Potential abuse of SMB services

### 4. SMB Vulnerability Scripts

- No specific MS vulnerabilities detected
- However, misconfiguration exists

## Analysis:

The SMB service is highly misconfigured, allowing anonymous access and user enumeration. This significantly increases the attack surface.

## Conclusion:

SMB is a high-risk attack vector due to weak access control and exposed user information.

---

## 11. Web Directory Enumeration (Gobuster)  
  
### Command Used:  

```bash  
gobuster dir -u http://192.168.241.131 -w /usr/share/wordlists/dirb/common.txt
```
### Key Findings:

#### 1. phpMyAdmin

- Database management panel exposed
- Potential for credential-based attack

#### 2. phpinfo.php

- Sensitive server information exposed
- Reveals PHP configuration and environment details

#### 3. WebDAV (/dav)

- Potential file upload vulnerability
- May allow remote code execution

#### 4. Additional Directories

- /test, /twiki identified
- Possible outdated or vulnerable applications

#### 5. Restricted Files

- .htaccess and .htpasswd detected (403)
- Indicates server configuration files exist

### Analysis:

The web server exposes multiple sensitive endpoints which can be leveraged for further attacks.

### Conclusion:

Web application presents multiple attack vectors including file upload, information disclosure, and credential attacks.

---

### 12. WebDAV File Upload Vulnerability

### Command Used:

```bash
cadaver http://192.168.241.131/dav
```

#### Location:
```
http://192.168.241.131/dav
```

#### Finding:
- WebDAV service allows file upload without authentication
- Uploaded file successfully accessed via browser

#### Proof of Concept:
- Uploaded a test file using cadaver
- File was accessible via browser

#### Impact:
- Allows attackers to upload arbitrary files
- Can lead to Remote Code Execution (RCE)
- Full system compromise possible

#### Severity:
High

#### Recommendation:
- Disable WebDAV if not required
- Restrict upload permissions
- Implement authentication
- Validate file types

---

### 13. SMB Share Permission Analysis

#### Tool Used:
```
smbmap
```

### Command Used:

```bash
smbmap -H 192.168.241.131
```

#### Finding:
- Share: tmp
- Permissions: READ, WRITE

#### Impact:
- Allows file upload and read access without restriction
- Can be used to store malicious files

#### Analysis:
The SMB share is misconfigured with excessive permissions.

#### Severity:
High

---

### 14. WebDAV Remote Code Execution (RCE)

### Command Used:

```bash
davtest -url http://192.168.241.131/dav

```

#### Location:
http://192.168.241.131/dav

#### Tool Used:
```
davtest
```

#### Findings:
- File upload allowed for multiple extensions
- PHP files successfully executed on server

#### Proof:
- Uploaded PHP file using WebDAV
- File execution confirmed via browser

#### Impact:
- Remote Code Execution (RCE)
- Attacker can run arbitrary commands
- Full system compromise possible

#### Severity:
Critical

#### Recommendation:
- Disable WebDAV if not required
- Restrict file upload permissions
- Disable script execution in upload directories

---

