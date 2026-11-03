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