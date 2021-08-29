# ScriptKidddie Website Investigation
Three input functions perform some server-side commands based on user input:

## NMAP
![[Pasted image 20210529150429.png]]
Nmap does work but can't do much with that as the input field won't take anything other than the required IP format. I tried to manipulate the input in Burp, but nothing seems to work

## Payloads
![[Pasted image 20210529202819.png]]
This one is more interesting as it takes a file upload. It has three options Windows, Linux and Android. When supplying input it generates a meterpreter payload so this must be using msfvemon or msfpayload to generate it on the back end. Will need to investigate this further.

## Searchsploit
![[Pasted image 20210529150950.png]]
This one searches the searchsploit db on the back end. I have had a play with the inputs but it only seems to search the db and I can't break out. I'll have a look at this in Burp and see if there's something more interesting here
