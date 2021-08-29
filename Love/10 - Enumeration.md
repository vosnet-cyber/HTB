# Enumeration
## NMAP
```sql
# Nmap 7.91 scan initiated Sat Jul 31 09:21:19 2021 as: nmap -sCV -vv -oA nmap/nmap_LoveScript 10.10.10.239
Nmap scan report for 10.10.10.239
Host is up, received reset ttl 127 (0.017s latency).
Scanned at 2021-07-31 09:21:19 BST for 24s
Not shown: 993 closed ports
Reason: 993 resets
PORT     STATE SERVICE      REASON          VERSION
80/tcp   open  http         syn-ack ttl 127 Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1j PHP/7.3.27)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: Voting System using PHP
135/tcp  open  msrpc        syn-ack ttl 127 Microsoft Windows RPC
139/tcp  open  netbios-ssn  syn-ack ttl 127 Microsoft Windows netbios-ssn
443/tcp  open  ssl/http     syn-ack ttl 127 Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
| ssl-cert: Subject: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in/localityName=norway/organizationalUnitName=love.htb/emailAddress=roy@love.htb
| Issuer: commonName=staging.love.htb/organizationName=ValentineCorp/stateOrProvinceName=m/countryName=in/localityName=norway/organizationalUnitName=love.htb/emailAddress=roy@love.htb
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-01-18T14:00:16
| Not valid after:  2022-01-18T14:00:16
| MD5:   bff0 1add 5048 afc8 b3cf 7140 6e68 5ff6
| SHA-1: 83ed 29c4 70f6 4036 a6f4 2d4d 4cf6 18a2 e9e4 96c2
| -----BEGIN CERTIFICATE-----
| MIIDozCCAosCFFhDHcnclWJmeuqOK/LQv3XDNEu4MA0GCSqGSIb3DQEBCwUAMIGN
| MQswCQYDVQQGEwJpbjEKMAgGA1UECAwBbTEPMA0GA1UEBwwGbm9yd2F5MRYwFAYD
| VQQKDA1WYWxlbnRpbmVDb3JwMREwDwYDVQQLDAhsb3ZlLmh0YjEZMBcGA1UEAwwQ
| c3RhZ2luZy5sb3ZlLmh0YjEbMBkGCSqGSIb3DQEJARYMcm95QGxvdmUuaHRiMB4X
| DTIxMDExODE0MDAxNloXDTIyMDExODE0MDAxNlowgY0xCzAJBgNVBAYTAmluMQow
| CAYDVQQIDAFtMQ8wDQYDVQQHDAZub3J3YXkxFjAUBgNVBAoMDVZhbGVudGluZUNv
| cnAxETAPBgNVBAsMCGxvdmUuaHRiMRkwFwYDVQQDDBBzdGFnaW5nLmxvdmUuaHRi
| MRswGQYJKoZIhvcNAQkBFgxyb3lAbG92ZS5odGIwggEiMA0GCSqGSIb3DQEBAQUA
| A4IBDwAwggEKAoIBAQDQlH1J/AwbEm2Hnh4Bizch08sUHlHg7vAMGEB14LPq9G20
| PL/6QmYxJOWBPjBWWywNYK3cPIFY8yUmYlLBiVI0piRfaSj7wTLW3GFSPhrpmfz0
| 0zJMKeyBOD0+1K9BxiUQNVyEnihsULZKLmZcF6LhOIhiONEL6mKKr2/mHLgfoR7U
| vM7OmmywdLRgLfXN2Cgpkv7ciEARU0phRq2p1s4W9Hn3XEU8iVqgfFXs/ZNyX3r8
| LtDiQUavwn2s+Hta0mslI0waTmyOsNrE4wgcdcF9kLK/9ttM1ugTJSQAQWbYo5LD
| 2bVw7JidPhX8mELviftIv5W1LguCb3uVb6ipfShxAgMBAAEwDQYJKoZIhvcNAQEL
| BQADggEBANB5x2U0QuQdc9niiW8XtGVqlUZOpmToxstBm4r0Djdqv/Z73I/qys0A
| y7crcy9dRO7M80Dnvj0ReGxoWN/95ZA4GSL8TUfIfXbonrCKFiXOOuS8jCzC9LWE
| nP4jUUlAOJv6uYDajoD3NfbhW8uBvopO+8nywbQdiffatKO35McSl7ukvIK+d7gz
| oool/rMp/fQ40A1nxVHeLPOexyB3YJIMAhm4NexfJ2TKxs10C+lJcuOxt7MhOk0h
| zSPL/pMbMouLTXnIsh4SdJEzEkNnuO69yQoN8XgjM7vHvZQIlzs1R5pk4WIgKHSZ
| 0drwvFE50xML9h2wrGh7L9/CSbhIhO8=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds syn-ack ttl 127 Windows 10 Pro 19042 microsoft-ds (workgroup: WORKGROUP)
3306/tcp open  mysql?       syn-ack ttl 127
| mysql-info: 
|_  MySQL Error: Host '10.10.14.4' is not allowed to connect to this MariaDB server
5000/tcp open  http         syn-ack ttl 127 Apache httpd 2.4.46 (OpenSSL/1.1.1j PHP/7.3.27)
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1j PHP/7.3.27
|_http-title: 403 Forbidden
Service Info: Hosts: www.example.com, LOVE, www.love.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h41m33s, deviation: 4h02m31s, median: 21m32s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 17217/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 46453/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 39089/udp): CLEAN (Timeout)
|   Check 4 (port 21885/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 10 Pro 19042 (Windows 10 Pro 6.3)
|   OS CPE: cpe:/o:microsoft:windows_10::-
|   Computer name: Love
|   NetBIOS computer name: LOVE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-07-31T01:43:10-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-31T08:43:11
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jul 31 09:21:43 2021 -- 1 IP address (1 host up) scanned in 24.02 seconds
```

Ok a few ports open:
80 - HTTP: Looks like a good place to start
135 - MSRPC: Standard Windows type port
139 - NetBIOS: Another standard Windows port
443 - HTTPS: Also looks like a good place to poke around, leaks username roy@love.htb, and domains: love.htb, staging.love.htb
445 - SMB: Very Windows, gives us some host info might be worth a look too for null auth etc. Might come in handy later
3306 - MYSQL: Unusual on a Windows box but as the script shows we can't login to it from my host machine. Is probably bound to localhost login only. I'm sure we'll be back here later though
5000 - HTTP?: Looks like another PHP web server on this port. Also gives us some host/domain information: www.example.com, LOVE, www.love.htb

First things first, let's get all of those web domains into the hosts file in case there is some virtual hosting stuff going on then we can dig into each one:
```bash
nano /etc/hosts

10.10.10.239    www.love.htb staging.love.htb love.htb
```

## Web Apps
### Port 80
#### love.htb
![[Pasted image 20210731095331.png]]
So, we have some login page here. We know maybe one user "roy" (but this needs a user ID so may not be a username) and we also know they are using an SQL database so maybe some SQL injection? Will also do a gobuster on this and as we know this is a PHP server we'll look for .php files too.
Ok more recon:
`gobuster dir -u http://love.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php -o gobuster/gob_love80`
```bash
grep "200\|301" gobuster/gob_love80
/includes             (Status: 301) [Size: 332] [--> http://love.htb/includes/]
/images               (Status: 301) [Size: 330] [--> http://love.htb/images/]
/admin                (Status: 301) [Size: 329] [--> http://love.htb/admin/]
/plugins              (Status: 301) [Size: 331] [--> http://love.htb/plugins/]
/index.php            (Status: 200) [Size: 4388]
/Admin                (Status: 301) [Size: 329] [--> http://love.htb/Admin/]
/Images               (Status: 301) [Size: 330] [--> http://love.htb/Images/]
/.                    (Status: 200) [Size: 4388]
/Includes             (Status: 301) [Size: 332] [--> http://love.htb/Includes/]
/ADMIN                (Status: 301) [Size: 329] [--> http://love.htb/ADMIN/]
/Index.php            (Status: 200) [Size: 4388]
/dist                 (Status: 301) [Size: 328] [--> http://love.htb/dist/]
/IMAGES               (Status: 301) [Size: 330] [--> http://love.htb/IMAGES/]
/tcpdf                (Status: 301) [Size: 329] [--> http://love.htb/tcpdf/]
/Plugins              (Status: 301) [Size: 331] [--> http://love.htb/Plugins/]
/INCLUDES             (Status: 301) [Size: 332] [--> http://love.htb/INCLUDES/]
/PlugIns              (Status: 301) [Size: 331] [--> http://love.htb/PlugIns/]
/INDEX.php            (Status: 200) [Size: 4388]
```

Urls to investigate:
/admin
![[Pasted image 20210731102110.png]]
Looks more like a user name type situation going on here. Will need to do some more checking around for password for Roy

/dist 
![[Pasted image 20210731102618.png]]
Had a quick look, it is what you'd expect with CSS and other bits, not too interesting but will come back if I hit a dead end

/images 
![[Pasted image 20210731102743.png]]
Yup, some images. Nothing obvious in them though

/includes 
![[Pasted image 20210731103056.png]]
PHP stuff, probably not much here but will have a look in a bit

/plugins
![[Pasted image 20210731103151.png]]
Unlikely to be useful but may have a look if running out of ideas

/tcpdf
![[Pasted image 20210731103239.png]]
I'm not sure what this is. I'll need to investigate further.

#### staging.love.htb
![[Pasted image 20210731181815.png]]
Ok this could be more interesting. Demo URL seems to scan files? Might be able to use this.


### Port 443
![[Pasted image 20210731095116.png]]
Ok not much here. We know for certain now it's a PHP server so can do a gobuster on this and look for .php files too.
```bash
gobuster dir -u https://love.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php -k -b 403 -o gobuster/gob_love443

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://love.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   403
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2021/07/31 10:11:09 Starting gobuster in directory enumeration mode
===============================================================
/examples             (Status: 503) [Size: 399]
                                               
===============================================================
2021/07/31 10:13:34 Finished
===============================================================
```

### Port 5000
![[Pasted image 20210731095603.png]]
Same as 443 so can do another gobuster maybe and see what we get.



### SMB
![[Pasted image 20210731095859.png]]
No null auth on this so can't list shares and user Roy is password protected as you'd expect. We'll come back to this.
