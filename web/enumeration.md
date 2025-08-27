# Site enumeration
## Nmap
<pre>nmap -Pn -sV -p- -sC IP</pre>

## website

Directory and file
<pre>gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.js</pre>

Subdomain
<pre>ffuf -u http://SITE/ -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.SITE" -fw 1</pre>

