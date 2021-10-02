# Getting User
## Web Application
Under the capture menu item you're sent to /data/1 and there is no captures there. If you change the URL to /data/0 there is data. Download that and inspect in wireshark.
![[Pasted image 20210725192255.png]]

### Getting the password
Examining in Wireshark you can see there is an FTP stream captured. As we know this is plaintext protocol just follow the TCP stream and located the username and password.
![[Pasted image 20210725192518.png]]
![[Pasted image 20210725192551.png]]

Now `get user.txt` and open it or cat it to get the flag: `10dfd10d3fd7a679e8314e95a8bef7ae`

I always like to see if the FTP server has been configured to restrict to the one directory. `cd ..` and we can get out of the directory and browse the whole filesystem. This isn't best practice as it now means an attacker could potentialy gain access to configurations files on the system.

Users often use the same password for other services on the system so let's try SSH with the creds we have for the FTP login:
![[Pasted image 20210725215442.png]]

Success!! We're now on the box as the Nathan user.

# Getting Root
Ok so let's take a look at what is running:  
![[Pasted image 20210730121110.png]]
Hmm, ok so nothing there. Ok another command I like to run when jumping on to a box is `sudo -l` to see if there's any commands we can run as root:
![[Pasted image 20210730141421.png]]
Nope, nothing...

Well we know there is a web server running so lets go take a look around for that. First place to try is the usual `/var/www/html` directory, and we find `app.py`. 
![[Pasted image 20210730121527.png]]
That could be interesting, let's take a closer look.

Here's the code:
```python
#!/usr/bin/python3

import os
from flask import *
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
import tempfile
import dpkt
from werkzeug.utils import append_slash_redirect

app = Flask(__name__)
app.config['TEMPLATES_AUTO_RELOAD'] = True
app.secret_key = b'\x81\x02&\x18\\a0ej\x06\xec\x917y*\x04Y\x83e\xebC\xee\xab\xcf\xac;\x8dx\x8bf\xc4\x15'
limiter = Limiter(app, key_func=get_remote_address, default_limits=["99999999999999999 per day", "99999999999999999999 per hour"])
pcapid = 0
lock = False

@app.before_first_request
def get_file_id():
        global pcapid
        path = os.path.join(app.root_path, "upload")
        onlyfiles = [f for f in os.listdir(path) if os.path.isfile(os.path.join(path, f))]
        ints = []
        for x in onlyfiles:
                try:
                        ints.append(int(x.replace(".pcap", "")))
                except:
                        pass
        try:
                pcapid = max(ints)+1
        except:
                pcapid = 0


def get_appid():
        global pcapid
        return pcapid

def increment_appid():
        global pcapid
        pcapid += 1

def get_lock():
        global lock
        while lock:
                pass
        lock = True

def release_lock():
        global lock
        lock = False

def process_pcap(pcap_path):
        reader = dpkt.pcap.Reader(open(pcap_path, "rb"))
        counter=0
        ipcounter=0
        tcpcounter=0
        udpcounter=0

        for ts, pkt in reader:
                counter+=1
                eth=dpkt.ethernet.Ethernet(pkt)

                try:
                        ip=dpkt.ip.IP(eth.data)
                except:
                        continue

                ipcounter+=1

                if ip.p==0:
                        tcpcounter+=1

                if ip.p==dpkt.ip.IP_PROTO_UDP:
                        udpcounter+=1

        data = {}
        data['Number of Packets'] = counter
        data['Number of IP Packets'] = ipcounter
        data['Number of TCP Packets']  = tcpcounter
        data['Number of UDP Packets']  = udpcounter
        return data


@app.route("/")
def index():
        return render_template("index.html")

PCAP_MAGIC_BYTES = [b"\xa1\xb2\xc3\xd4", b"\xd4\xc3\xb2\xa1", b"\x0a\x0d\x0d\x0a"]

@app.route("/capture")
@limiter.limit("10 per minute")
def capture():

        get_lock()
        pcapid = get_appid()
        increment_appid()
        release_lock()

        path = os.path.join(app.root_path, "upload", str(pcapid) + ".pcap")
        ip = request.remote_addr
        # permissions issues with gunicorn and threads. hacky solution for now.
        #os.setuid(0)
        #command = f"timeout 5 tcpdump -w {path} -i any host {ip}"
        command = f"""python3 -c 'import os; os.setuid(0); os.system("timeout 5 tcpdump -w {path} -i any host {ip}")'"""
        os.system(command)
        #os.setuid(1000)

        return redirect("/data/" + str(pcapid))

@app.route("/ip")
def ifconfig():
	d = os.popen("ifconfig").read().strip()
	print(d)
	return render_template("index.html", rawtext=d)

@app.route("/netstat")
def netstat():
	d = os.popen("netstat -aneop").read().strip()
	print(d)
	return render_template("index.html", rawtext=d)

@app.route("/data")
def data():
        if "data" not in session:
                return redirect("/")
        data = session.pop("data")
        path = session.pop("path")
        return render_template("data.html", data=data, path=path)

@app.route("/data/<id>")
def data_id(id):
        try:
                id = int(id)
        except:
                return redirect("/")
        try:
                data = process_pcap(os.path.join(app.root_path, "upload", str(id) + ".pcap"))
                path = str(id) + ".pcap"
                return render_template("index.html", data=data, path=path)
        except Exception as e:
                print(e)
                return redirect("/")

@app.route("/download/<id>")
def download(id):
        try:
                id = int(id)
        except:
                return redirect("/")
        uploads = os.path.join(app.root_path, "upload")
        return send_from_directory(uploads, str(id) + ".pcap", as_attachment=True)

if __name__ == "__main__":
        app.run("0.0.0.0", 80, debug=True)
```

Looking through this code we see this little nugget:
![[Pasted image 20210730122050.png]]

This is interesting, so it looks like there is a problem capturing the network packets as `tcpdump` needs to be run as root and the application is not running with sufficient privileges. Or there is just some other problem with threading? Either way what they have done is set the UserID to 0 (root) and then run the command to capture the data they want. 

Now, normally the `setuid()` command can only be used by the root user so for it to work inside this script either the script/app has to be running as root or something else is going on. Because the comments in the script say they're using a "hacky" way to get this working makes me think there is something else going on here. 

## Possible Rabbit-hole 
Now I think most people when they see this might think "ok, let's run this app and see what happens" which is fine and I would do that too most of the time however, I didn't do this until after I got root, but we'll come back to why this was the case in a moment. So, this is where you may fall down a small rabbit-hole, if you run this app you'll get an error because it tries to use port 80 which is already in use, so you can just change it to another port e.g. 1500. When you browse to your version of the app you'll see it probably errored and that you can run python commands if you enter the pin which is displayed on the command line:
![[Pasted image 20210730134856.png]]

You may also notice that it gives the same PIN each time you run it so you may think ah-ha! Maybe I can get to the debug console on the main app, it might be running as root because it's on port 80. When you do a bit of research about the debug console you'll see if it is enabled you can browse to `/console` and maybe get access to system through that and use the same PIN codes. If not the same PIN codes maybe you can figure out the code by running some scripts you can find online. But this doesn't work as there is no debug console available on the running app, so that's a dead end anyway.

----

So, back to why I didn't mess around with the app straight way. I was looking at the code where it captures the network packets and thought "this is a Python script, why would you need to run another instance of Python from within the code to `setuid(0)` and run the command you want if the app was already running as root?" Then the thought hit me "they need a hacky way to capture packets, if the app was running as root it wouldn't need to be a hack, so the app doesn't run as root!" 

Ok so now I'm fairly certain the app isn't running as root so how does this work because you shouldn't be able to use `setuid()`, so... and BOOM! "Linux Capabilites!!" popped into my head; what if the Python binary has the `CAP_SETUID` setting enabled then it could be used to set the UID to 0 and run system commands as root! So let's take a look:
![[Pasted image 20210730142456.png]]
Yes, and it has the Effective, Inherited and Permitted flags set so as far as I understand it I can just run Python from the command line and get it to execute bash as root and we can just use the command in the code to do just that. Let's try:
![[Pasted image 20210730143306.png]]

Sweet! now we're root we can grab the flag from `/root/root.txt`.







