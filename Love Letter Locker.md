A THM Room
By Sovren
Part of the Love at First Breach 2026 Event

# Reconnaissance 

Made an account with abc:password credentials and made a new letter.

# Enumeration

### Nmap

`sudo nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.80.175.82 -p-`

`PORT     STATE SERVICE VERSION`
`22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)`
`| ssh-hostkey:` 
`|   256 07:30:6a:97:f5:94:84:44:d1:32:21:39:8e:96:ae:5a (ECDSA)`
`|_  256 f8:07:82:f9:34:84:ff:dd:dd:d6:1b:0d:fb:f7:ed:fa (ED25519)`
`5000/tcp open  http    Werkzeug httpd 3.1.5 (Python 3.12.3)`
`|_http-title: Home | LoveLetter Locker`
`Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Adtran 424RG FTTH gateway (92%), Android 9 - 10 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%)`
`No exact OS matches for host (test conditions non-ideal).`
`Network Distance: 3 hops`
`Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel`

---
### Gobuster

`gobuster dir -u http://10.80.175.82:5000 -w /usr/share/wordlists/dirb/common.txt`

`letters              (Status: 302) [Size: 231] [--> /login?next=%2Fletters]`
`login                (Status: 200) [Size: 1017]`
`logout               (Status: 302) [Size: 229] [--> /login?next=%2Flogout]`
`register             (Status: 200) [Size: 1048]`

---

`gobuster dir -u http://10.80.175.82:5000/letters -w /usr/share/wordlists/dirb/common.txt`

`new                  (Status: 302) [Size: 243] [--> /login?next=%2Fletters%2Fnew]`

---

# Exploitation

This room is an example of IDOR (Insecure Direct Object Reference)

By making an account and creating a new letter, we could see how the site stored letters.

Before creating a new letter, we could see that there were already 2 letters in the archive. 

By changing /letter/3 to /letter/2 we were able to exploit the site and view data through IDOR that would otherwise remain hidden to a general user. 

The flag is found in /letter/1 
`THM{1_c4n_r3ad_4ll_l3tters_w1th_th1s_1d0r}`