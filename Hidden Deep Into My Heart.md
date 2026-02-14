# A THM Room 
# By Sovren
# Part of the Love at First Breach 2026 Event


My Dearest Hacker,

Cupid's Vault was designed to protect secrets meant to stay hidden forever. Unfortunately, Cupid underestimated how determined attackers can be.

Intelligence indicates that Cupid may have unintentionally left vulnerabilities in the system. With the holiday deadline approaching, you've been tasked with uncovering what's hidden inside the vault before it's too late.

You can find the web application here: `http://<target.ip>:5000`

---
# Reconnaissance

'Love Letters Anonymous

Welcome to our secret valentine message board!

Share your anonymous love letters with the world...'

-No immediate low hanging fruit after inspecting the page source material. 

--Moving onto the enumeration stage.


---
# Enumeration

###### NMAP

`nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.81.170.97 -p-`

`PORT     STATE SERVICE VERSION`
`22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)`
`| ssh-hostkey:` 
`|   256 34:f0:9d:41:30:13:81:d6:96:e9:40:bc:a7:86:f0:0e (ECDSA)`
`|_  256 47:60:5b:14:97:96:d7:8d:0d:e6:b4:d8:51:15:b9:47 (ED25519)`
`5000/tcp open  http    Werkzeug httpd 3.1.5 (Python 3.10.12)`
`|_http-title: Love Letters Anonymous`
`| http-robots.txt: 1 disallowed entry` 
`|_/cupids_secret_vault/*`
`Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Adtran 424RG FTTH gateway (92%), Android 9 - 10 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%)`
`No exact OS matches for host (test conditions non-ideal).`
`Network Distance: 3 hops`
`Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel`

-An Nmap scan reveals cupid's secret vault. There is nothing immediately obvious from a quick inspect. Next step is to enumerate with Gobuster. 


---
###### GOBUSTER

`gobuster dir -u http://10.81.170.97:5000 -w /usr/share/wordlists/dirb/common.txt`

`console              (Status: 400) [Size: 167]`
`robots.txt           (Status: 200) [Size: 70]`

---

`gobuster dir -u http://10.81.170.97:5000/cupids_secret_vault -w /usr/share/wordlists/dirb/common.txt`

 `administrator        (Status: 200) Size: 2381` 


---

-The /administrator page contains a login form. The hydra syntax should take into account the following below:

--name="username"

--name="password"

Our first Hydra attempt:

`hydra -l admin  -P /usr/share/wordlists/rockyou.txt -vV -s 5000 10.81.170.97 http-post-form "/cupids_secret_vault/administrator:username=^USER^&password=^PASS^:Invalid credentials\!"`

rockyou.txt is a massive password file and we might do better with a shorter list. 

`hydra -l admin  -P /usr/share/wordlists/seclists/Common-Credentials/10k-most-common.txt -vV -s 5000 10.81.170.97 http-post-form "/cupids_secret_vault/administrator:username=^USER^&password=^PASS^:Invalid credentials\!"`

admin, administrator and cupid have both failed here. 

---

###### /robots.txt

the /robots.txt file contains:

`User-agent: *`
`Disallow: /cupids_secret_vault/*`

`cupid_arrow_2026!!!`

No need for Hydra in the end. 

admin:cupid_arrow_2026!!!

A successful login with these credentials gives us the flag to complete the room

 `THM{l0v3_is_in_th3_r0b0ts_txt}` 
