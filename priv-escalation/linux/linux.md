## Enumeration

**`hostname`** - name of the device, can be usefull <br>
**`uname -a`** - information about kernel system, search in exploit DB <br>
**`cat /proc/version`** - information about compilers (c++), kernel version <br>
**`ps`** - idk <br>
**`id`** - users privilege level<br>
**`sudo -l`** - what commands can user run with root privileges <br>
**`env`** - enviromental variables<br>
**`cat /etc/passwd`** - discoved users on the system <br>
**`cat /etc/shadow`** - hashes of passwords <br>
**`cat /etc/issue`** - what linux is running <br>
**`history`** - can show passwords and usernames or any info <br> <br>

### find commands
`find . -name flag1.txt 2>/dev/null` find the file named “flag1.txt” in the current directory <br>
`find /home -name flag1.txt` find the file names “flag1.txt” in the /home directory <br>
`find / -type d -name config` find the directory named config under “/” <br>
`find / -type f -perm 0777` find files with the 777 permissions (files readable, writable, and executablebyall users) <br>
`find / -perm a=x` find executable files <br>
`find /home -user frank` find all files for user “frank” under “/home” <br>
`find / -mtime 10` find files that were modified in the last 10 days <br>
`find / -atime 10` find files that were accessed in the last 10 day <br>
`find / -cmin -60` find files changed within the last hour (60 minutes) <br>
`find / -amin -60` find files accesses within the last hour (60 minutes) <br>
`find / -size 50M` find files with a 50 MB size <br>

## Kernel exploit

Check the kernel information and try to find existing exploits, be sure you understande them. [ExploitDB](https://www.exploit-db.com/)

## Sudo

Run `sudo -l` command. Than look at [GTFOBins](https://gtfobins.github.io/) and search for bypass ways.

## SUID
Commands with permmisions of the file owner <br>

`find / -type f -perm -04000 -ls 2>/dev/null` will list files that have SUID or SGID bits set. <br>
Again you can compare compare executables on [GTFOBins+suid](https://gtfobins.github.io/#+suid).

It can be usefull to crack passwords using /etc/shadow and /etc/passwd files `unshadow passwd.txt shadow.txt > passwords.txt` and then johnTheRipper `john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt passwords.txt` <br>

## Capabilities

We can use the `getcap -r / 2>/dev/null` tool to list enabled capabilities. <br>
Search on [GTFOBins+capabilities](https://gtfobins.github.io/#+capabilities) <br>

## Cron Jobs

