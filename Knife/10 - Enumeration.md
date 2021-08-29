# NMAP
```sql
# Nmap 7.91 scan initiated Tue Aug 17 11:21:05 2021 as: nmap -sCV -v -n -oA nmap/nmap_knifeInital 10.10.10.242
Nmap scan report for 10.10.10.242
Host is up (0.020s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
|   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
|_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title:  Emergent Medical Idea
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 17 11:21:13 2021 -- 1 IP address (1 host up) scanned in 7.49 seconds
```

Ok two ports open, let's do a full port scan to be sure.
```sql
# Nmap 7.91 scan initiated Tue Aug 17 11:21:29 2021 as: nmap -p- -v -n -oA nmap/nmap_knifeFullTCP 10.10.10.242
Nmap scan report for 10.10.10.242
Host is up (0.018s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Read data files from: /usr/bin/../share/nmap
# Nmap done at Tue Aug 17 11:21:40 2021 -- 1 IP address (1 host up) scanned in 10.81 seconds
```

I'll do a top 100 UDP too just in case.
```sql
# Nmap 7.91 scan initiated Tue Aug 17 11:29:22 2021 as: nmap --top-ports=100 -sU -v -n -oA nmap/nmap_knifeTop100UDP 10.10.10.242
Nmap scan report for 10.10.10.242
Host is up (0.014s latency).
All 100 scanned ports on 10.10.10.242 are closed

Read data files from: /usr/bin/../share/nmap
# Nmap done at Tue Aug 17 11:30:57 2021 -- 1 IP address (1 host up) scanned in 95.33 seconds
```
Nothing there.

Yup just two ports then. 
22:SSH - Not going to do much with that just yet as we'll need creds, but this does show us it's an Ubuntu box
80:Apache Webserver - Ok let's go see what we have here then


# Webserver - Port 80
![[Pasted image 20210817113346.png]]

Ok looking at this page there is nothing here to navigate to. What appears to be links at the top are just text and don't link to anything. Looking at the source code there doesn't seem to be much there either. Let's try a bit of the usual recon for robots.txt and other usual website pages and also run a gobuster on the site.

## GoBuster
```bash
gobuster dir -u http://10.10.10.242/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -o gobuster_root                                                            
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.242/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/08/18 12:14:40 Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 277]
/.htm                 (Status: 403) [Size: 277]
/.                    (Status: 200) [Size: 5815]
/.htaccess            (Status: 403) [Size: 277]
/.htc                 (Status: 403) [Size: 277]
/.html_var_DE         (Status: 403) [Size: 277]
/server-status        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.html.               (Status: 403) [Size: 277]
/.html.html           (Status: 403) [Size: 277]
/.htpasswds           (Status: 403) [Size: 277]
/.htm.                (Status: 403) [Size: 277]
/.htmll               (Status: 403) [Size: 277]
/.html.old            (Status: 403) [Size: 277]
/.ht                  (Status: 403) [Size: 277]
/.html.bak            (Status: 403) [Size: 277]
/.htm.htm             (Status: 403) [Size: 277]
/.html1               (Status: 403) [Size: 277]
/.htgroup             (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.html.LCK            (Status: 403) [Size: 277]
/.html.printable      (Status: 403) [Size: 277]
/.htm.LCK             (Status: 403) [Size: 277]
/.htaccess.bak        (Status: 403) [Size: 277]
/.htx                 (Status: 403) [Size: 277]
/.htmls               (Status: 403) [Size: 277]
/.htlm                (Status: 403) [Size: 277]
/.htm2                (Status: 403) [Size: 277]
/.html-               (Status: 403) [Size: 277]
/.htuser              (Status: 403) [Size: 277]
```
Nothing obvious going on here so let's take a deeper look under the hood.