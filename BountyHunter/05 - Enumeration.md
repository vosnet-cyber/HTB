# NMAP
```bash
# Nmap 7.91 scan initiated Sat Aug 21 20:48:05 2021 as: nmap -sCV -v -n -oA nmap/nmap_Initial 10.10.11.100
Nmap scan report for 10.10.11.100
Host is up (0.017s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Aug 21 20:48:13 2021 -- 1 IP address (1 host up) scanned in 7.79 seconds
```

OK, two ports open:
22 - SSH: Probably not going to use this right now but does let us know this is a Linux Ubuntu box.
80 - HTTP: Looks like this will be where we start on this box. We also know it's an Apache web server too.

I'll also run a full TCP port scan too, just in case.
```bash
# Nmap 7.91 scan initiated Sat Aug 21 20:52:09 2021 as: nmap -p- -v -n -oA nmap/nmap_FullTCP 10.10.11.100
Nmap scan report for 10.10.11.100
Host is up (0.026s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Sat Aug 21 20:52:24 2021 -- 1 IP address (1 host up) scanned in 14.59 seconds
```
Nope, nothing else to find here. If there's nothing to find on the webserver I may also run a UDP scan too but I doubt that will be needed for this box.

# Web Server - Port 80
![[Pasted image 20210821205419.png]]
So the "About" and "Contact Us" links only go to ID's on the main page however looking at the hovering over the "Portal" menu item we can see it goes to `portal.php`. This is useful to know as we can do some extra enumeration and search for `.php` extensions too.

Let's get a gobuster going and then go look at the Portal page.
## GoBuster
```bash
===============================================================                                               
Gobuster v3.1.0                                                                                                                                                       by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                                                                                         ===============================================================                                                                                                        [+] Url:                     http://10.10.11.100/                                                                                                                      [+] Method:                  GET                                                                                                                                        [+] Threads:                 10                                                                                                                                        [+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/raft-small-words.txt                   
[+] Negative Status codes:   404                                                                                                                                  
[+] User Agent:              gobuster/3.1.0                                                                                                                          
[+] Extensions:              php                                                                                                                                      
[+] Timeout:                 10s                                                                                                                                      
===============================================================                                                                                                      
2021/08/21 20:59:29 Starting gobuster in directory enumeration mode                                                                                                  
===============================================================                                                                                                      
/.html                (Status: 403) [Size: 277]                                                                                                                      
/.php                 (Status: 403) [Size: 277]                                                                                                                      
/.html.php            (Status: 403) [Size: 277]                                                                                                                     
/js                   (Status: 301) [Size: 309] [--> http://10.10.11.100/js/]                                                                                        
/index.php            (Status: 200) [Size: 25169]                                                                                                                    
/css                  (Status: 301) [Size: 310] [--> http://10.10.11.100/css/]                                                                                        
/.htm                 (Status: 403) [Size: 277]                                                                                                                      
/.htm.php             (Status: 403) [Size: 277]                                                                                                                      
/db.php               (Status: 200) [Size: 0]                                        
/assets               (Status: 301) [Size: 313] [--> http://10.10.11.100/assets/]    
/resources            (Status: 301) [Size: 316] [--> http://10.10.11.100/resources/] 
/.                    (Status: 200) [Size: 25169]                                    
/portal.php           (Status: 200) [Size: 125]                               
```
`db.php` stands out here as a 0 lengh file. As it's a `.php` file it probably has PHP code in the file but that isn't presented here as PHP code is processed before the page is rendered in the browser or web request. Let's keep this in the back of our mind for now as this may become a target for us once we have access to the box.

Most of the other directories are redirected and forbidden so we can't easily get into those but they may not contain anything we need anyway as the titles suggest things like JavaScript, CSS, and Assets which are usually images.

## /resources
![[Pasted image 20210821210925.png]]
This directory has listing enabled and there's an interesting README.txt file in here. This could come in handy to remember later on as I haven't seen anything yet that this really corresponds to.

## Portal.php
![[Pasted image 20210821210132.png]]
OK this just provides a link to another page `log_submit.php`:
![[Pasted image 20210821210227.png]]

Let's give it a quick test and see what happens in the page:
![[Pasted image 20210822034734.png]]

Ok, so it just returns what we enter into the fields. This might be interesting as anytime you process user input and return it to the user there is potential for abuse. We'll give this a very quick test to see if there's anything unusual or abusable:
+ SQL Injection: `'- --` - Nope that was returned no problem
+ Special Character: `!"£$%^&*()_-+={}[]'@~#/?,.>` - Nothing seemed to do anything apart from `<` this did seem to break the output