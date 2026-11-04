# Task

The third server is an MX and management server for the internal network. Subsequently, this server has the function of a backup server for the internal accounts in the domain. Accordingly, a user named `HTB` was also created here, whose credentials we need to access.

---

>Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer.



## Attack summary



---

## Nmap scan 

```
sudo nmap  10.129.14.94 -sC -sV --open 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-03 05:48 EST
Nmap scan report for 10.129.14.94
Host is up (0.071s latency).
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
|_  256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: CAPA AUTH-RESP-CODE UIDL TOP USER RESP-CODES STLS PIPELINING SASL(PLAIN)
|_ssl-date: TLS randomness does not represent time
143/tcp open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: Pre-login more have ID capabilities listed SASL-IR post-login STARTTLS AUTH=PLAINA0001 OK ENABLE LOGIN-REFERRALS LITERAL+ IDLE IMAP4rev1
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_ssl-date: TLS randomness does not represent time
993/tcp open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_imap-capabilities: Pre-login more ID have listed SASL-IR post-login capabilities IMAP4rev1 OK LOGIN-REFERRALS ENABLE LITERAL+ IDLE AUTH=PLAINA0001
995/tcp open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=NIXHARD
| Subject Alternative Name: DNS:NIXHARD
| Not valid before: 2021-11-10T01:30:25
|_Not valid after:  2031-11-08T01:30:25
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA TOP USER RESP-CODES SASL(PLAIN) AUTH-RESP-CODE PIPELINING UIDL
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.07 seconds

```

 ```
 nmap -p- 10.129.202.20 --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-04 03:33 EST
Nmap scan report for 10.129.202.20
Host is up (0.054s latency).
Not shown: 65255 closed tcp ports (reset), 275 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE
22/tcp  open  ssh
110/tcp open  pop3
143/tcp open  imap
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 22.11 seconds
 ```

```
telnet 10.129.202.20 110                         
Trying 10.129.202.20...
Connected to 10.129.202.20.
Escape character is '^]'.
+OK Dovecot (Ubuntu) ready.
HELO
-ERR Unknown command.
LIST *
-ERR Unknown command.
LIST
-ERR Unknown command.
-ERR Too many invalid bad commands.
Connection closed by foreign host.
                                                                                                                    
┌──(kali㉿kali)-[~/labs/footprinting/hard]
└─$ telnet 10.129.202.20 143
Trying 10.129.202.20...
Connected to 10.129.202.20.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ STARTTLS AUTH=PLAIN] Dovecot (Ubuntu) ready.
a1 LIST "" *
a1 BAD Error in IMAP command received by server.
LIST "" *
LIST BAD Error in IMAP command received by server.


```

## snmp enum

```
nmap -sU 10.129.202.20 -T4                     
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-04 03:38 EST
Warning: 10.129.202.20 giving up on port because retransmission cap hit (6).
Stats: 0:01:28 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 11.36% done; ETC: 03:51 (0:11:27 remaining)
Stats: 0:03:25 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 22.47% done; ETC: 03:53 (0:11:47 remaining)
Stats: 0:07:45 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 47.11% done; ETC: 03:54 (0:08:42 remaining)
Stats: 0:11:10 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 66.73% done; ETC: 03:54 (0:05:34 remaining)
Stats: 0:13:37 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 80.74% done; ETC: 03:55 (0:03:15 remaining)
Stats: 0:17:10 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 99.99% done; ETC: 03:55 (0:00:00 remaining)
Nmap scan report for 10.129.202.20
Host is up (0.055s latency).
Not shown: 991 closed udp ports (port-unreach)
PORT      STATE         SERVICE
68/udp    open|filtered dhcpc
161/udp   open          snmp
2048/udp  open|filtered dls-monitor
2343/udp  open|filtered nati-logos
16832/udp open|filtered unknown
21131/udp open|filtered unknown
21207/udp open|filtered unknown
21609/udp open|filtered unknown
44185/udp open|filtered unknown

```

```
nmap -sU 10.129.202.20 -T4 -p 161 --script snmp-info
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-04 03:59 EST
Nmap scan report for 10.129.202.20
Host is up (0.063s latency).

PORT    STATE SERVICE
161/udp open  snmp
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: 5b99e75a10288b6100000000
|   snmpEngineBoots: 10
|_  snmpEngineTime: 29m55s


```

## brute forcing community string

```
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt -w 100 10.129.202.20
Scanning 1 hosts, 3219 communities
10.129.202.20 [backup] Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64

```

```
snmpwalk -v2c -c backup 10.129.202.20 
iso.3.6.1.2.1.1.1.0 = STRING: "Linux NIXHARD 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (248787) 0:41:27.87
iso.3.6.1.2.1.1.4.0 = STRING: "Admin <tech@inlanefreight.htb>"
iso.3.6.1.2.1.1.5.0 = STRING: "NIXHARD"
iso.3.6.1.2.1.1.6.0 = STRING: "Inlanefreight"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (26) 0:00:00.26
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (249950) 0:41:39.50
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 EA 0B 04 09 0A 33 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.4.0-90-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro ipv6.disable=1 maybe-ubiquity
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 158
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
iso.3.6.1.2.1.25.1.7.1.1.0 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.2.6.66.65.67.75.85.80 = STRING: "/opt/tom-recovery.sh"
iso.3.6.1.2.1.25.1.7.1.2.1.3.6.66.65.67.75.85.80 = STRING: "tom NMds732Js2761"
iso.3.6.1.2.1.25.1.7.1.2.1.4.6.66.65.67.75.85.80 = ""
iso.3.6.1.2.1.25.1.7.1.2.1.5.6.66.65.67.75.85.80 = INTEGER: 5
iso.3.6.1.2.1.25.1.7.1.2.1.6.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.7.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.2.1.20.6.66.65.67.75.85.80 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.2.1.21.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.3.1.1.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
iso.3.6.1.2.1.25.1.7.1.3.1.2.6.66.65.67.75.85.80 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:
Authentication token manipulation error
chpasswd: (line 1, user tom) password not changed
Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.3.1.3.6.66.65.67.75.85.80 = INTEGER: 4
iso.3.6.1.2.1.25.1.7.1.3.1.4.6.66.65.67.75.85.80 = INTEGER: 1
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.1 = STRING: "chpasswd: (user tom) pam_chauthtok() failed, error:"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.2 = STRING: "Authentication token manipulation error"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.3 = STRING: "chpasswd: (line 1, user tom) password not changed"
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = STRING: "Changing password for tom."
iso.3.6.1.2.1.25.1.7.1.4.1.2.6.66.65.67.75.85.80.4 = No more variables left in this MIB View (It is past the end of the MIB tree)
                   
```

```
telnet 10.129.202.20 143             
Trying 10.129.202.20...
Connected to 10.129.202.20.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ STARTTLS AUTH=PLAIN] Dovecot (Ubuntu) ready.
LOGIN tom NMds732Js2761
LOGIN BAD First parameter in line is IMAP's command tag, not the command name. Add that before the command, like: a login user pass
1a LOGIN tom NMds732Js2761
1a OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE SORT SORT=DISPLAY THREAD=REFERENCES THREAD=REFS THREAD=ORDEREDSUBJECT MULTIAPPEND URL-PARTIAL CATENATE UNSELECT CHILDREN NAMESPACE UIDPLUS LIST-EXTENDED I18NLEVEL=1 CONDSTORE QRESYNC ESEARCH ESORT SEARCHRES WITHIN CONTEXT=SEARCH LIST-STATUS BINARY MOVE SNIPPET=FUZZY PREVIEW=FUZZY LITERAL+ NOTIFY SPECIAL-USE] Logged in


```