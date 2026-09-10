---
title: "venus"
date: 2026-03-08T14:46:04+08:00
draft: false
description: "HackMyVM 靶机 venus 的渗透测试与提权记录"
categories: ["靶机复盘"]
tags: ["HackMyVM"]
---
说明：

你好，黑客，欢迎来到 HMVLab 第一章：金星！这是一个面向初学者的 CTF 闯关练习，你可以在这里锻炼 Linux 和 CTF 相关技能，
那我们开始吧！:)
首先，每个用户的家目录都在 /pwned/用户名 路径下，
里面会有一个名为 mission.txt 的文件，里面写着你需要完成的任务，
完成任务就能获取下一个用户的密码。
目录里还会有 flagz.txt 文件，
如果你在 https://hackmyvm.eu 注册过账号，可以提交这个文件内容参与排行榜（可选）。
为了增加趣味性，这里还有隐藏关卡和隐藏 flag：D
大部分目录你都没有写入权限，所以如果你需要编写脚本或文件，
请使用 /tmp 目录，注意这个目录会被定期清空……
最后（同样重要）：
部分用户可以修改 /www 目录下的文件，这些文件可以通过 http://venus.hackmyvm.eu 访问。所以如果你拿到一个能修改 /www/hi.txt 的用户权限，
你写入的内容会显示在 http://venus.hackmyvm.eu/hi.txt 上。
如果你有疑问、想法或想反馈问题，可以加入我们的 Discord：
https://discord.gg/DxDFQrJ
记住还有其他人在玩，请保持礼貌。
黑客技术，快乐玩耍！

## level1

关卡说明：用户 **sophia** 把她的密码保存在了本目录下的一个**隐藏文件**里。找到它，并以 sophia 身份登录

```bash
ls -al
total 40
drwxr-x--- 2 root   hacker 4096 Apr  5  2024 .
drwxr-xr-x 1 root   root   4096 Apr  5  2024 ..
-rw-r----- 1 root   hacker   31 Apr  5  2024 ...
-rw-r--r-- 1 hacker hacker  220 Apr 23  2023 .bash_logout
-rw-r--r-- 1 hacker hacker 3526 Apr 23  2023 .bashrc
-rw-r----- 1 root   hacker   16 Apr  5  2024 .myhiddenpazz
-rw-r--r-- 1 hacker hacker  807 Apr 23  2023 .profile
-rw-r----- 1 root   hacker  287 Apr  5  2024 mission.txt
-rw-r----- 1 root   hacker 2542 Apr  5  2024 readme.txt

file .myhiddenpazz
.myhiddenpazz: ASCII text

cat .myhiddenpazz
Y1o645M3mR84ejc
```

## level2

关卡说明：用户 **angela** 把她的密码保存在了一个文件里，但她不记得放在哪儿了…… 她只记得文件名是 **whereismypazz.txt**。

```bash
find / -name whereismypazz.txt 2>/dev/null
/usr/share/whereismypazz.txt

cat /usr/share/whereismypazz.txt
oh5p9gAABugHBje
```

## level3

关卡说明：用户 **emma** 的密码，在文件 **findme.txt** 的第 **4069 行**。

```bash
awk 'NR==4069' findme.txt
fIvltaGaq0OUH8O

sed -n  '4069p' findme.txt
# -n 不能显示所有行 4069p 只打印4069行

head -n 4069 findme.txt | tail -n 1
```

## level4

关卡说明：用户 **mia** 把她的密码留在了名为 **-** 的文件里。

```bash
cat ./-
iKXIYg0pyEH2Hos

cat < -
```

## level5

关卡说明：用户 **camila** 似乎把她的密码留在了名为 **hereiam** 的文件夹里。

```bash
find / -name hereiam 2>/dev/null
/opt/hereiam

ls -al /opt/hereiam
total 12
drwxr-xr-x 2 root root 4096 Apr  5  2024 .
drwxr-xr-x 1 root root 4096 Apr  5  2024 ..
-rw-r--r-- 1 root root   16 Apr  5  2024 .here

cd /opt/hereiam

file .here
.here: ASCII text

cat .here
F67aDmCAAgOOaOc
```

## level6

关卡说明：用户 **luna** 把她的密码留在了 **muack** 文件夹里的一个文件中。

```bash
find . -type f -exec file {} \; | grep "ASCII text"
./111/111/muack: ASCII text

cat ./111/111/muack
j3vkuoKQwvbhkMc
```

## level7

关卡说明：用户 **eleanor** 把密码留在了一个**大小为 6969 字节**的文件里。

```bash
find / -size 6969c -type f 2>/dev/null
/usr/share/moon.txt

cat /usr/share/moon.txt
UNDchvln6Bmtu7b
```

## level8

关卡说明：用户 **victoria** 把密码留在了一个**属主为用户 violin** 的文件里。

```bash
find / -user violin 2>/dev/null
/usr/local/games/yo

cat /usr/local/games/yo
pz8OqvJBFxH0cSj
```

## level9

zip的解压方法：

```bash
# 1. unzip（最常用）
unzip data.zip

# 2. 查看内容（不解压）
unzip -l data.zip

# 3. 解压到指定目录
unzip data.zip -d /tmp/output
```

关卡说明：用户 **isla** 把她的密码留在了一个 **zip 压缩包** 里。

```bash
file passw0rd.zip
passw0rd.zip: Zip archive data, at least v1.0 to extract, compression method=store

unzip passw0rd.zip -d /tmp/tmp
Archive:  passw0rd.zip
extracting: /tmp/tmp/pwned/victoria/passw0rd.txt  
 
cat /tmp/tmp/pwned/victoria/passw0rd.txt 
D3XTob0FUImsoBb
```

## level10

```bash
1. 截取单个字符

echo "hello world" | cut -c 1
# 输出：h（第1个字符）

echo "hello world" | cut -c 7
# 输出：w（第7个字符）
2. 截取范围

echo "hello world" | cut -c 1-5
# 输出：hello（第1到第5个字符）

echo "hello world" | cut -c 7-11
# 输出：world
3. 从某位置到结尾

echo "hello world" | cut -c 7-
# 输出：world（从第7个到结尾）
4. 从开头到某位置

echo "hello world" | cut -c -5
# 输出：hello（从开头到第5个）
5. 多个位置

echo "hello world" | cut -c 1,3,5
# 输出：hlo（第1、3、5个字符）

echo "hello world" | cut -c 1-3,7-9
# 输出：helwor（第1-3和第7-9）
对文件操作：

# 文件内容：
# apple
# banana
# cherry

cut -c 1-3 file.txt
# 输出：
# app
# ban
# che
```

关卡说明：用户 **violet** 的密码，在**以 a9HFX 开头的那一行**里（这 5 个字符不算密码本身）。

```bash
grep -E "^a9HFX.*" passy | cut -c 6-
WKINVzNQLKLDVAc
```

## level11

关卡说明：用户 **lucy** 的密码在**以 0JuAZ 结尾的那一行**中（最后这 5 个字符不属于密码）

```bash
grep -E ".*?0JuAZ$" end | cut -c -15
OCmMUjebG53giud
```

## level12

关卡说明：The password of the user elena is between the characters fu and ck

```bash
grep
-a
# 当作文本处理
-o
# 只输出匹配的部分
-P
# 使用 Perl 兼容的正则表达式

# \K 表示：匹配前面的内容，但不包含在最终结果中。

语法	      名称	   含义
(?=...)	   正向前瞻	   后面必须跟着 a，但不包含 a 在匹配结果中
(?!...)	   负向前瞻	   后面不能是...
(?<=...)   正向后顾	   前面必须是...
(?<!...)   负向后顾	   前面不能是...
```

做法：

```bash
grep -oP "fu\K.*?(?=ck)" file.yo
# 4xZ5lIKYmfPLg9t
```

## level13

关卡说明：The user alice has her password is in an environment variable.

```bash
cat /proc/self/environ | tr '\0' '\n' | grep -i "pass"
PASS=Cgecy2MY2MWbaqt
```

## level14

关卡说明: The admin has left the password of the user anna as a comment in the file passwd.

```
/etc/passwd 一共 7 个字段，用 : 分开

    用户名
    密码占位（x）
    UID
    GID
    注释（GECOS）
    家目录
    Shell
```

做法：

```bash
grep -oP "^(?:[^:]*:){4}\K[^:]*" /etc/passwd
root
daemon
bin
sys
sync
games
man
lp
mail
news
uucp
proxy
www-data
backup
Mailing List Manager
ircd
nobody
systemd Network Management
MySQL Server,,,
systemd Time Synchronization
w8NvY27qkpdePox
```

最后一个就是密码。

```
(?:...) 是非捕获组
^ 在开头 匹配输入字符串的开头位置；在中括号中，不匹配中括号内的内容。
```

## level15

关卡说明：Maybe sudo can help you to be natalia.

```bash
# 查看当前用户可以用 sudo 执行什么命令
sudo -l

# 输出格式:
# (目标用户) 权限要求: 允许的命令
# 例如:
# (natalia) NOPASSWD: /bin/bash  - 可以无密码切换到natalia并执行bash
# (ALL) ALL                      - 可以以任何用户身份执行任何命令
# (root) /usr/bin/vim            - 只能以root身份执行vim

sudo -u <用户名> <命令>
# 以指定用户的身份执行命令

# -i: 模拟登录 shell (类似 su -)
sudo -i                  # 切换到 root 的登录环境
sudo -u natalia -i       # 切换到 natalia 的登录环境

# -s: 启动 shell
sudo -s                  # 启动 root 的 shell
sudo -u natalia -s       # 启动 natalia 的 shell

# -E: 保留当前环境变量
sudo -E command

# -k: 清除 sudo 的时间戳(下次需要重新输入密码)
sudo -k

# -v: 延长 sudo 的时间戳(刷新密码有效期)
sudo -v
```

```bash
sudo -l  # 看看当前用户能执行啥命令

Matching Defaults entries for anna on venus:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User anna may run the following commands on venus:
    (natalia) NOPASSWD: /bin/bash
    
sudo -u natalia /bin/bash
```

## level16

关卡说明：The password of user eva is encoded in the base64.txt file

```bash
base64 -d base64.txt
upsCA3UFu10fDAO
```

## level17

关卡说明：The password of the clara user is found in a file modified on May 1, 1968.

```bash
# 查找在特定日期之后修改的文件
find /path -newermt "2024-01-01"
find /path -newermt "2024-01-01 10:30:00"

# 查找在特定日期之前修改的文件
find /path ! -newermt "2024-01-01"


参数	   含义	说明
-mtime   n	   恰好 n 天前	n*24小时前到(n+1)*24小时前之间
-mtime  -n	   n 天之内	最近 n 天内修改的文件
-mtime  +n	   n 天之前	超过 n 天前修改的文件

# 1. 查找最近 24 小时内修改的文件
find /path -mtime 0

# 2. 查找最近 7 天内修改的文件
find /path -mtime -7
# 包括: 0天前, 1天前, 2天前, ..., 6天前
```

这道题的时间给的有问题，应该是1970年。

```bash
find / ! -newermt "1970-5-1" -type f 2>/dev/null
/proc/69046/task/69046/fdinfo/6
/proc/69046/fdinfo/5
/usr/lib/cmdo

cat /usr/lib/cmdo
39YziWp5gSvgQN9

ls -al /usr/lib/cmdo
-rw-r--r-- 1 root root 16 Jan  1  1970 /usr/lib/cmdo

find / -type f -mtime +20440 2>/dev/null
/usr/lib/cmdo
```

## level18

关卡说明：The password of user frida is in the password-protected zip (rockyou.txt can help you) 

```bash
fcrackzip -u #使用unzip 测试密码
-D # 使用字典攻击模式
-p # 指定字典攻击模式
```

```bash
scp -P 5000 clara@venus.hackmyvm.eu:~/protected.zip ~/Desktop

fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt protected.zip

PASSWORD FOUND!!!!: pw == pass123

unzip -P pass123 protected.zip -d /tmp/1.txt
Archive:  protected.zip
extracting: /tmp/1.txt/pwned/clara/protected.txt
 
cat /tmp/1.txt/pwned/clara/protected.txt
Ed4ErEUJEaMcXli
```

## level19

关卡说明：The password of eliza is the only string that is repeated (unsorted) in repeated.txt.

```
uniq -d repeated.txt
Fg6b6aoksceQqB9
```

## level20

关卡说明：The user iris has left me her key.

```bash
scp -P 5000 eliza@venus.hackmyvm.eu:~/.iris_key ~/Desktop

chmod 600 ./.iris_key

ssh -i ./.iris_key iris@venus.hackmyvm.eu -p 5000


# 密码：kYjyoLcnBZ9EJdz
```

## level21

关卡说明：User eloise has saved her password in a particular way. 

```bash
base64 -d eloise                 
����JFIF``���ExifMM;sML�J���
                           >������85��85�
                                       �2021:11:10 10:18:032021:11:10 10:18:03sML��
                                       
# 看头，发现其是照片。

mkdir /tmp/tm
cd /tmp/tm
touch 1.jpg
base64 -d eloise > /tmp/tm/1.jpg

scp -P 5000 iris@venus.hackmyvm.eu:/tmp/tm/1.jpg ~/Desktop


yOUJlV0SHOnbSPm
```

## level22

关卡说明：User lucia has been creative(有创意) in saving her password.

```bash
cat hi
00000000: 7576 4d77 4644 5172 5157 504d 6547 500a
# 发现其是16进制
cat hi | xxd -r 
uvMwFDQrQWPMeGP
# xxd 将2 进制转换为16进制数据
# -r 16 -> 2
```

## level23

关卡说明：The user isabel has left her password in a file in the /etc/xdg folder but she does not remember the name, however she has dict.txt that can help her to remember.

```bash
#!/bin/bash
while read path;do

        cat "/etc/xdg/${path}" 2>/dev/null

done < ~/dict.txt
# 1.sh
```

```bash
sh 1.sh
H5ol8Z2mrRsorC0
```

## level24

关卡说明：The password of the user freya is the only string that is not repeated(重复、重叠) in different.txt

```bash
sort different.txt | uniq -u 
EEDyYFDwYsmYawj
```

## level25

关卡说明：User alexa puts her password in a .txt file in /free every minute and then deletes it.

`$? 是 Bash 的特殊变量，存储上一个命令的退出码（exit code）。`

1.sh

```bash
#!/bin/bash
#
while true;do
        result=$(cat /free/*.txt 2>/dev/null)
        if [ $? -eq 0 ]; then
                echo "$result"
                break
        else
                echo "no file"
        fi
done
```

```bash
sh 1.sh
no file
no file
no file
no file
no file
no file
no file
no file
no file
no file
no file
no file
mxq9O3MSxxX9Q3S
```

## level26

关卡说明：The password of the user ariel is online! (HTTP)

```bash
curl http://localhost
33EtHoz9a0w2Yqo
```

## level27

关卡说明：Seems that ariel dont save the password for lola, but there is a temporal file.

.goas.swp 是Vim 的交换文件（swap file），是 Vim 编辑器的临时文件。

```bash
file .goas.swp
.goas.swp: Vim swap file, version 8.2, pid 2299, user teste, host deb11, file ~teste/goas, modified

mkdir /tmp/t12

vim -r .goas.swp 

:w /tmp/t12/goas.txt

cat /tmp/t12/goas.txt

Thats my little DIc with my old and current passw0rds:

-->ppkJjqYvSCIyAhK
-->cOXlRYXtJWnVQEG
--rxhKeFKveeKqpwp
-->RGBEMbZHZRgXZnu
-->IaOpTdAuhSjGZnu
-->NdnszvjulNellbK
-->GBUguuSpXVjpxLc
-->rSkPlPhymYcerMJ
-->PEOppdOkSqJZweH
-->EKvJoTBYlwtwFmv
-->d3LieOzRGX5wud6   密码
-->mYhQVLDKdJrsIwG
-->DabEJLmAbOQxEnD
-->LkWReDaaLCMDlLf
-->cbjYGSvqAsqIvdg
-->QsymOOVbzSaKmRm
-->bnQgcXYamhSDSff
-->VVjqJGRrnfKmcgD
```

## level28

关卡说明：The user celeste has left a list of names of possible .html pages where to find her password.

```bash
curl -s # 静默模式，不显示
grep -q # 静默搜索，返回退出码，不加-q 不返回退出码。
```

```bash
mkdir /tmp/1

vim 1.sh
#!/bin/bash

while read filename; do
        result=$(curl -s "http://localhost/${filename}.html")
        if ! echo "${result}" | grep -q "404"; then
                echo "${filename}.html"
                echo "$result"
                echo "===== end ====="
        fi
done < pages.txt

:wq

sh 1.sh
Real-Time Communication.html

===== end =====
cebolla.html
VLSNMTKwSV2o8Tn
===== end =====
```

## level29(mysql)

关卡说明：The user celeste has access to mysql but for what?用户 Celeste 拥有访问 MySQL 数据库的权限，但她这么做的目的是什么呢？

1. 服务管理命令

```bash
# 启动 MySQL
sudo systemctl start mysql
sudo systemctl start mysqld
sudo service mysql start

# 停止 MySQL
sudo systemctl stop mysql
sudo systemctl stop mysqld

# 重启 MySQL
sudo systemctl restart mysql
sudo systemctl restart mysqld

# 查看状态
sudo systemctl status mysql
sudo systemctl status mysqld

# 开机自启
sudo systemctl enable mysql

# 禁用开机自启
sudo systemctl disable mysql

# 重新加载配置
sudo systemctl reload mysql
```

2. 连接命令

```bash
# 基本连接
mysql -u root -p
mysql -u username -p
mysql -u username -pPassword123  # 不推荐，密码会显示

# 指定主机和端口
mysql -h localhost -u root -p
mysql -h 127.0.0.1 -P 3306 -u root -p
mysql -h remote.server.com -u root -p

# 连接并指定数据库
mysql -u root -p database_name

# 使用 socket 文件连接
mysql -u root -p -S /var/run/mysqld/mysqld.sock

# 使用配置文件连接
mysql --defaults-file=/path/to/my.cnf
```

3. 执行 SQL 命令

```bash
# 执行单条 SQL
mysql -u root -p -e "SHOW DATABASES;"

# 执行多条 SQL
mysql -u root -p -e "USE mydb; SELECT * FROM users; SHOW TABLES;"

# 从文件执行 SQL
mysql -u root -p < script.sql
mysql -u root -p database_name < backup.sql

# 执行并输出到文件
mysql -u root -p -e "SELECT * FROM users;" > output.txt

# 静默模式（不显示表格边框）
mysql -u root -p -s -e "SELECT * FROM users;"

# 批处理模式
mysql -u root -p --batch -e "SELECT * FROM users;"
```

4.备份和恢复

```bash
# 备份单个数据库
mysqldump -u root -p database_name > backup.sql

# 备份所有数据库
mysqldump -u root -p --all-databases > all_backup.sql

# 备份多个数据库
mysqldump -u root -p --databases db1 db2 db3 > backup.sql

# 备份单个表
mysqldump -u root -p database_name table_name > table_backup.sql

# 备份表结构（不含数据）
mysqldump -u root -p --no-data database_name > structure.sql

# 只备份数据（不含结构）
mysqldump -u root -p --no-create-info database_name > data.sql

# 压缩备份
mysqldump -u root -p database_name | gzip > backup.sql.gz

# 恢复数据库
mysql -u root -p database_name < backup.sql
mysql -u root -p < all_backup.sql

# 恢复压缩备份
gunzip < backup.sql.gz | mysql -u root -p database_name
```

这关有问题了，不知道为啥。

```
nina   ixpeqdWuvC5N9kG
```

```sql
mysql -uceleste -pVLSNMTKwSV2o8Tn
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.11.6-MariaDB-0+deb12u1 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.        

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| venus              |
+--------------------+
2 rows in set (0.004 sec)

MariaDB [(none)]> use venus
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [venus]> show tables;
+-----------------+
| Tables_in_venus |
+-----------------+
| people          |
+-----------------+
1 row in set (0.001 sec)

MariaDB [venus]> select column_name form information_schema.columns where table_name="people";
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'information_schema.columns where table_name="people"' at line 1        
MariaDB [venus]> select column_name from information_schema.columns where ta
ble_name="people";
+-------------+
| column_name |
+-------------+
| id_people   |
| uzer        |
| pazz        |
+-------------+
3 rows in set (0.006 sec)

MariaDB [venus]> select * from people
    -> ;
+-----------+---------------+--------------------------------+
| id_people | uzer          | pazz                           |
+-----------+---------------+--------------------------------+
|         1 | nuna          | ixpfdsvcxeqdW                  |
|         2 | nona          | ixpvcxvcxeqdW                  |
|         3 | manue         | ixpfdsfdseqdW                  |
|         4 | samoa         | ixperrewrweqdW                 |
|         5 | dsaewq        | ixpefdsfsqdW                   |
|         6 | fdsfewrew     | ixpedvcxv4qdW                  |
|         7 | koiuoiudsadas | ixpredsfdeqdW                  |
|         8 | vcxfdsfew     | ixp342432eqdW                  |
|         9 | dasd          | ixpeiuyiuyqdW                  |
|        10 | helen         | uytuytjhgixpeqdW               |
|        11 | tudou         | ijhgjghxpeqfdfsfddW            |
|        12 | fdsoiurew     | ixpfdsfsdsvcxvcxeqdW           |
|        13 | inan          | imbnmnbxpeqdW                  |
|        14 | zkret         | ixpeqkjhkhjkdW                 |
|        15 | cjhcx         | i432423xpeqdW                  |
|        16 | sfdfdsml      | ixpeqdsfsdfdsfW                |
|        17 | svcxvcxml     | 432423ixpeqdW                  |
|        18 | xml           | ixpejhgjhgqdW                  |
|        19 | pdf           | ixperewrewrewqdW               |
|        20 | txt           | ixpeuytuytqdW                  |
|        21 | vcxvcx        | ixpefdsfdsfdfdsqdW             |
|        22 | dsadsa        | ixpeqdjhkjhW                   |
|        23 | lel           | ixpvcxvcxvcxeqdW               |
|        24 | lul           | ixpeqdmnbmnbmnbmW              |
|        25 | dog           | ixperewrewrewqdW               |
|        26 | cat           | ixvcxvdsfsdvpeqdW              |
|        27 | pet           | ixiufohsyuoirewpeqdW           |
|        28 | pzzz          | ixvcxvcxvpeqdW                 |
|        29 | ls            | ixpehgfdhdhqdW                 |
|        30 | vi            | ixpetrvvrqdW                   |
|        31 | tmux          | iuovcxoiujvcxixpeqdW           |
|        32 | screen        | ixpeqrewregfdgdW               |
|        33 | yes           | ixpebvcgdfgqdW                 |
|        34 | nop           | ixpefdsqdW                     |
|        35 | haha          | 8===xKmPDsJSKpHLzkqKXyjx===D~~ |
|        36 | love          | ixpegfdgqdW                    |
|        37 | dsadsa        | fdsvcxvcxixpeqdW               |
|        38 | d4t4          | erwerewreixpeqdW               |
|        39 | nna           | gdfgdixpeqdW                   |
|        40 | nin           | aaafdixpeqdW                   |
|        41 | tre           | fdsafixpeqdW                   |
|        42 | tfas          | igfdgfdgxpeqdW                 |
|        43 | zcxc          | ixfdgdfgpeqdW                  |
|        44 | yuio          | ixpgbvcbvcbeqdW                |
|        45 | jhgyurtrt     | treterterixpeqdW               |
|        46 | lodsa         | itreterxpeqdW                  |
|        47 | zarah         | ixpvcbvcbeqdW                  |
|        48 | zkkad         | ixpedfgvbcxbvcqdW              |
|        49 | bvher         | vcxvcxgfdgfdixpeqdW            |
|        50 | dsadsa        | ixpeqergdfwer32dW              |
|        51 | ch4rm         | ixpeewf23qdW                   |
|        52 | Aza           | ixpjhgjgheqdW                  |
|        53 | avij          | ixpegfdgdfgqdW                 |
|        54 | crom          | ixpefdbvvcbrqdW                |
|        55 | bubu          | ixpetretretqdW                 |
|        56 | bebe          | ixpeghfgfdqdW                  |
|        57 | baba          | ixpeffesfqdW                   |
|        58 | bael          | ixpesdvsdvsdqdW                |
|        59 | vaze          | ixpe23r23rf23qdW               |
|        60 | upper         | ixpe43r43rqdW                  |
|        61 | loz           | ixpeqddfsdW                    |
|        62 | mind          | ixpfsdfsdfsdeqdW               |
|        63 | mymy          | ixpevcxvqdW                    |
|        64 | ina           | ixpee23e32rqdW                 |
|        65 | ein           | ixpejytjytjhgjqdW              |
|        66 | n1n4          | ixpehgjghjhghgqdW              |
|        67 | where         | ixljkgjgpeqdW                  |
|        68 | you           | ixpeqdhggjhgjW                 |
|        69 | are           | ixVCXVCXVCXVCXdW               |
|        70 | what          | ixpeqhgjggdW                   |
|        71 | dsaqqqqqq     | ixpeqVCXVCXdW                  |
|        72 | h0j3n         | ixpemnbmbnmghqdW               |
|        73 | nana          | ixpeqVSDFWCdW                  |
|        74 | nina          | ixpeqdWuvC5N9kG                |
|        75 | nunu          | ixpeSFDSFDSVCXqdW              |
|        76 | fdse          | ixpeDFSWEF2qdW                 |
|        77 | dsar          | ixpeF43F3F34qdW                |
|        78 | yop           | ixpeqdWCSDFDSFD                |
|        79 | loco          | ixpeF43F34F3qdW                |
|        80 | zaza          | ixpeYUTHNYGTHYTqdW             |
|        81 | jhon          | ixpeFDSJYTUJTYqdW              |
|        82 | tell          | ixpeHYTTqdW                    |
|        83 | ma            | uyixptje4FSFWEFqdW             |
|        84 | mum           | jghixpeqdW                     |
|        85 | nanaa         | 432432ixpeqdW                  |
|        86 | nnnniinn      | irewxpeqdW                     |
|        87 | iourewoiure   | rewixpeqdW                     |
|        88 | lkjfdsoiu     | dsaixpeqdW                     |
|        89 | vcxnoj        | dasdasixpeqdW                  |
|        90 | ioyuwer       | ixpeqdvcxvcxW                  |
|        91 | kaka          | ixpeqdW                        |
|        92 | nini          | ixpeqdvcxW                     |
|        93 | zong          | ixpeqdWfdsfsdf                 |
|        94 | nana          | ixpefdsafdsqdW                 |
|        95 | ninna         | ixpeqOPUIFDSFDSdW              |
+-----------+---------------+--------------------------------+
95 rows in set (0.019 sec)

MariaDB [venus]> select * from people where length(pazz)=15;
+-----------+----------+-----------------+
| id_people | uzer     | pazz            |
+-----------+----------+-----------------+
|        16 | sfdfdsml | ixpeqdsfsdfdsfW |
|        44 | yuio     | ixpgbvcbvcbeqdW |
|        54 | crom     | ixpefdbvvcbrqdW |
|        58 | bael     | ixpesdvsdvsdqdW |
|        74 | nina     | ixpeqdWuvC5N9kG |
|        77 | dsar     | ixpeF43F3F34qdW |
|        78 | yop      | ixpeqdWCSDFDSFD |
|        79 | loco     | ixpeF43F34F3qdW |
+-----------+----------+-----------------+
8 rows in set (0.004 sec)

select concat(uzer:pazz) from people where length(pazz)=15;

ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ':pazz) from people where length(pazz)=15' at line 1
MariaDB [venus]> select concat(uzer,0x3a,pazz) from people where length(pazz
)=15;
+--------------------------+
| concat(uzer,0x3a,pazz)   |
+--------------------------+
| sfdfdsml:ixpeqdsfsdfdsfW |
| yuio:ixpgbvcbvcbeqdW     |
| crom:ixpefdbvvcbrqdW     |
| bael:ixpesdvsdvsdqdW     |
| nina:ixpeqdWuvC5N9kG     |
| dsar:ixpeF43F3F34qdW     |
| yop:ixpeqdWCSDFDSFD      |
| loco:ixpeF43F34F3qdW     |
+--------------------------+
8 rows in set (0.001 sec)
```

hydra 爆破如下字典：-C 可以爆破如下格式的字典

```
sfdfdsml:ixpeqdsfsdfdsfW
yuio:ixpgbvcbvcbeqdW
crom:ixpefdbvvcbrqdW
bael:ixpesdvsdvsdqdW
nina:ixpeqdWuvC5N9kG
dsar:ixpeF43F3F34qdW
yop:ixpeqdWCSDFDSFD
loco:ixpeF43F34F3qdW
```

```bash
hydra -C password.txt ssh://venus.hackmyvm.eu:5000
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-06 17:28:44
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries, ~1 try per task
[DATA] attacking ssh://venus.hackmyvm.eu:5000/
[5000][ssh] host: venus.hackmyvm.eu   login: nina   password: ixpeqdWuvC5N9kG                                                       
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-03-06 17:28:56

```

密码: `[5000][ssh] host: venus.hackmyvm.eu   login: nina   password: ixpeqdWuvC5N9kG                                                       `

```bash
cut -d: -f1 dict.txt > users.txt

cut           # 文本切割工具
-d:           # delimiter（分隔符）是冒号 :
-f1           # field（字段）取第1个字段
dict.txt      # 输入文件
> users.txt   # 输出重定向到文件

cut -d: -f2 dict.txt > passes.txt
cut           # 文本切割工具
-d:           # 分隔符是冒号 :
-f2           # 取第2个字段
dict.txt      # 输入文件
> passes.txt  # 输出到文件
```

## level30

关卡说明：The user kira is hidding something in http://localhost/method.php  

```bash
curl -s -i -X PUT http://localhost/method.php
HTTP/1.1 200 OK
Server: nginx/1.22.1
Date: Wed, 04 Mar 2026 11:57:04 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive


tPlqxSKuT4eP3yr
```

## level31

关卡说明：The user veronica visits a lot http://localhost/waiting.php

```bash
curl http://localhost/waiting.php

Im waiting for the user-agent PARADISE.

curl -A "PARADISE" http://localhost/waiting.php

QTOel6BodTx2cwX
```

## level32

关卡说明：The user veronica uses a lot the password from lana, so she created an alias.

```bash
alias
alias lanapass='UWbc0zNEVVops1v'
alias ls='ls --color=auto'
```

## level33

关卡说明：The user noa loves to compress her things.

```bash
mkdir /tmp/4

cp zip.gz /tmp/4

cd /tmp/4

file zip.gz
zip.gz: POSIX tar archive (GNU) # 打包

tar -xf zip.gz

cat ./pwned/lana/zip
9WWOPoeJrq6ncvJ

```

## level34

关卡说明：The password of maia is surrounded(包裹) by trash surrounded

```bash
file trash
trash: data

strings trash
b;pK
*&dv
 |.-
wsG9
D55-
\|gu
1q#^
YV!)}
f}nP
T735
5GOj'
g3-5v)S~hK
{Xu7
O;rTl,
]Bokc
04`0
X:Uf
;Vtr3
`vr)
k`      I
<(;pQ
@$LiJ
u7TI
*Q{r%
;%gzDB
b%/*
3g?d
=I+"
xfFN
\nh1hnDPHpydEjoEN
!       2L~8
JmN8
@%`j
,       ^,
e&xvN2
_cKn
.c|0
)|hd&
hl(p
fEr:
OdBb
?OsP
dnN9
J7e(
JL6(
wI;%vz
apPD
a5qi
|otr
4TTm
toyi
*f|F
.%J`t

# 密码：h1hnDPHpydEjoEN
```

## level35

关卡说明：The user gloria has forgotten the last 2 characters of her password ... They only remember that they were 2 lowercase letters.

```python
import string

letter = string.ascii_lowercase
password = "v7xUVE2e5bjUc"
with open('password.txt','w') as f:
    for i in letter:
        for j in letter:
            f.write(password + i + j + '\n')

```

字典制造。

```bash
hydra -l gloria -P ./password.txt ssh://venus.hackmyvm.eu:5000      
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-05 17:55:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 676 login tries (l:1/p:676), ~43 tries per task
[DATA] attacking ssh://venus.hackmyvm.eu:5000/
[STATUS] 149.00 tries/min, 149 tries in 00:01h, 532 to do in 00:04h, 11 active
[STATUS] 147.67 tries/min, 443 tries in 00:03h, 238 to do in 00:02h, 11 active
[STATUS] 146.00 tries/min, 584 tries in 00:04h, 97 to do in 00:01h, 11 active
[5000][ssh] host: venus.hackmyvm.eu   login: gloria   password: v7xUVE2e5bjUcxw
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 5 final worker threads did not complete until end.
[ERROR] 5 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-03-05 18:00:09

```

## level36

关卡说明：User alora likes drawings, that's why she saved her password as ... 

```
cat image

##########################################################
##########################################################
##########################################################
##########################################################
########              ##########  ##              ########
########  ##########  ##    ##  ####  ##########  ########
########  ##      ##  ##  ##  ######  ##      ##  ########
########  ##      ##  ####  ########  ##      ##  ########
########  ##      ##  ##        ####  ##      ##  ########
########  ##########  ##        ####  ##########  ########
########              ##  ##  ##  ##              ########
########################  ####  ##########################
########    ##  ####    ####  ##  ##      ##    ##########
############    ######  ##    ##      ##          ########
########    ##    ##  ##  ##            ####  ##  ########
##############      ##  ##    ######  ##    ####  ########
############    ##      ##  ########    ##  ##  ##########
########################    ####    ##  ##  ####  ########
########              ##    ####            ##  ##########
########  ##########  ######  ##########  ####  ##########
########  ##      ##  ####  ##      ######        ########
########  ##      ##  ##    ##  ######  ##  ####  ########
########  ##      ##  ####          ##    ##  ##  ########
########  ##########  ##      ####  ##  ##################
########              ##  ##                    ##########
##########################################################
##########################################################
##########################################################
##########################################################
```

这竟然是一个二维码。
密码：mhrTFCoxGoqUxtw

## level37

关卡说明：The user julie has created an iso with her password.

**文件说明**
music.iso - ISO 9660 格式的光盘镜像文件
包含了一张 CD-ROM 的完整内容
可以挂载或刻录到光盘

```bash
# 如果你在 WSL (Windows Subsystem for Linux) 中
sudo mkdir -p /mnt/cdrom  #
sudo mount -o loop music.iso /mnt/cdrom
# mount 挂载命令 将文件系统连接到目录树的某个位置
# -o 选项 loop 回环设备选项  -o loop 相当于把文件模拟成一个硬盘或光驱
# music.iso 要挂载的ISO镜像
# /mnt/cdrom ISO要显示的目录

# 查看内容
ls -la /mnt/cdrom

# 复制文件
cp -r /mnt/cdrom/* ./music_files/

# 卸载
sudo umount /mnt/cdrom

```

```bash
scp -P 5000 alora@venus.hackmyvm.eu:~/music.iso ~/Desktop

mkdir /tmp/temporary

sudo mount -o loop music.iso /tmp/temporary

cd /tmp/temporary

unzip music.zip

cat pwned/alora/music.txt 
sjDf4i2MSNgSvOv
```

## level38(diff)

关卡说明：The user irene believes that the beauty is in the difference.

```bash
diff 1.txt 2.txt
174c174
< 8VeRLEFkBpe2DSD # 密码
---
> aNHRdohjOiNizlU
```

## level39

关卡说明：The user adela has lent(lend 借给，提供) her password to irene.

```
mkdir /tmp/8

openssl pkeyutl -decrypt -inkey ./id_rsa.pem -in pass.enc -out /tmp/8/pass.txt

cat /tmp/8/pass.txt
nbhlQyKuaXGojHx
```

扩展：

```bash
openssl pkeyutl -decrypt -inkey ./id_rsa.pem -in pass.enc -out /tmp/8/pass.txt

1. openssl
OpenSSL 工具，用于加密/解密、证书管理等
2. pkeyutl
Public Key Utility（公钥工具）
用于公钥/私钥操作（加密、解密、签名、验证）
替代旧的 rsautl 命令
3. -decrypt
解密模式
使用私钥解密数据
4. -inkey ./id_rsa.pem
指定私钥文件
id_rsa.pem
 是当前目录下的私钥
5. -in pass.enc
输入文件：要解密的加密文件
6. -out /tmp/8/pass.txt
输出文件：解密后的明文保存位置
```

加密过程：

```bash
# 使用公钥加密文件
openssl pkeyutl -encrypt -pubin -inkey public_key.pem -in plaintext.txt -out encrypted.enc
# -pubin 表示输入的密钥文件是公钥

# 或使用私钥提取公钥后加密
openssl rsa -in id_rsa.pem -pubout -out public_key.pem
openssl pkeyutl -encrypt -pubin -inkey public_key.pem -in message.txt -out message.enc
```

## level40

关卡说明：User sky has saved her password to something that can be listened to.

```
cat wtf
.--. .- .--. .- .--. .- .-. .- -.. .. ... .

Cyber 解密：
PAPAPARADISE
papaparadise
```

## level41

关卡说明：User sarah uses header in http://localhost/key.php

```bash
curl -H "Key: true" http://localhost/key.php
LWOHeRgmIxg7fuS
```

## level42

关卡说明：The password of mercy is hidden in this directory.

```bash
ls -al
total 32
drwxr-x---  2 root  sarah 4096 Apr  5  2024 .
drwxr-xr-x 55 root  root  4096 Apr  5  2024 ..
-rw-r-----  1 root  sarah   16 Apr  5  2024 ...
-rw-r--r--  1 sarah sarah  220 Apr 23  2023 .bash_logout
-rw-r--r--  1 sarah sarah 3526 Apr 23  2023 .bashrc
-rw-r--r--  1 sarah sarah  807 Apr 23  2023 .profile
-rw-r-----  1 root  sarah   31 Apr  5  2024 flagz.txt
-rw-r-----  1 root  sarah  175 Apr  5  2024 mission.txt
cat < ...
ym5yyXZ163uIS8L
cat ./...
ym5yyXZ163uIS8L
```

## level43

关卡说明: User mercy is always wrong with the password of paula.

```bash
ls -al
total 32
drwxr-x---  2 root  mercy 4096 Apr  5  2024 .
drwxr-xr-x 55 root  root  4096 Apr  5  2024 ..
-rw-r-----  1 root  mercy  133 Apr  5  2024 .bash_history
-rw-r--r--  1 mercy mercy  220 Apr 23  2023 .bash_logout
-rw-r--r--  1 mercy mercy 3526 Apr 23  2023 .bashrc
-rw-r--r--  1 mercy mercy  807 Apr 23  2023 .profile
-rw-r-----  1 root  mercy   31 Apr  5  2024 flagz.txt
-rw-r-----  1 root  mercy  190 Apr  5  2024 mission.txt

cat .bash_history
ls -A
ls
rm /
ps
sudo -l
watch tv
vi /etc/logs
su paula
dlHZ6cvX6cLuL8p
history
history -c
logout
ssh paula@localhost
cat .
ls
ls -l
```

## level44

关卡说明：The user karla trusts me, she is part of my group of friends.

```bash
groups
# 查看当前用户的组
paula hidden

find / -group hidden 2>/dev/null
/usr/src/.karl-a

cat /usr/src/.karl-a
gYAmvWY3I7yDKRf
```

## level45

关卡说明：User denise has saved her password in the image.

`exiftool` 是查看和编辑文件元数据（`metadata`）的工具。

```
什么是元数据
元数据 = 文件的附加信息

照片元数据包括：
- 拍摄时间
- 相机型号
- GPS 位置
- 作者/版权
- 注释/描述
- 软件信息
等等
```

```bash
# 查看所有元数据
exiftool image.jpg

# 查看特定字段
exiftool -Comment image.jpg
exiftool -Author image.jpg
exiftool -Description image.jpg
```

```bash
exiftool yuju.jpg
ExifTool Version Number         : 13.25
File Name                       : yuju.jpg
Directory                       : .
File Size                       : 33 kB
File Modification Date/Time     : 2026:03:06 13:50:24+08:00
File Access Date/Time           : 2026:03:06 13:50:24+08:00
File Inode Change Date/Time     : 2026:03:06 13:50:24+08:00
File Permissions                : -rw-r-----
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 96
Y Resolution                    : 96
Exif Byte Order                 : Big-endian (Motorola, MM)
Artist                          : sML
Date/Time Original              : 2021:11:01 10:34:51
Create Date                     : 2021:11:01 10:34:51
Sub Sec Time Original           : 95
Sub Sec Time Digitized          : 95
XP Author                       : sML
Padding                         : (Binary data 2060 bytes, use -b option to extract)
XMP Toolkit                     : Image::ExifTool 12.16
About                           : pFg92DpGucMWccA
Creator                         : sML
Image Width                     : 442
Image Height                    : 463
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 442x463
Megapixels                      : 0.205
Create Date                     : 2021:11:01 10:34:51.95
Date/Time Original              : 2021:11:01 10:34:51.95
```

## level46

关卡说明：The user zora is screaming doas!

```bash
find / -name doas 2>/dev/null
/usr/share/doc/doas
/usr/bin/doas
/etc/pam.d/doas

file /usr/share/doc/doas
/usr/share/doc/doas: directory

file /usr/bin/doas
/usr/bin/doas: setuid ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=07ac0f9d39f13db413cc7c2df32463a66f8f6e6e, for GNU/Linux 3.2.0, stripped  

file /etc/pam.d/doas
/etc/pam.d/doas: ASCII text

cat /etc/pam.d/doas
#%PAM-1.0

# Set up user limits from /etc/security/limits.conf.      
session    required   pam_limits.so

@include common-auth
@include common-account
@include common-session-noninteractive

doas
usage: doas [-Lns] [-C config] [-u user] command [args]

doas -u zora /bin/bash
doas (denise@venus) password:
zora@venus:/pwned/denise$ 
```

密码：`BWm1R3jCcb53riO`

## level47

关卡说明：The user belen has left her password in venus.hmv

```
curl http://venus.hmv
2jA0E8bQ4WrGwWZ
```

## level48

关卡说明：It seems that belen has stolen the password of the user leona...

```bash
cat stolen.txt
$1$leona$lhWp56YnWAMz6z32Bw53L0
```

```
哈希格式解析
$1$leona$lhWp56YnWAMz6z32Bw53L0

$1$          = MD5-crypt 算法标识
leona        = salt（盐值）
lhWp56...L0  = 哈希值
```

```
哈希类型识别
$1$  = MD5-crypt (Unix)
$2$  = Blowfish (bcrypt)
$5$  = SHA-256
$6$  = SHA-512
$y$  = yescrypt
```

```bash
hashcat -m 500 -a 0 stolen.txt /usr/share/wordlists/rockyou.txt
-m 模式 500 MD5-crtpt
-a 0 字典模式
stolen.txt 要攻击的文件
rockyou.txt 字典

$1$leona$lhWp56YnWAMz6z32Bw53L0:freedom 
```

## level49

关卡说明：User ava plays a lot with the DNS of venus.hmv lately...

```bash
/etc/bind/ 是 BIND DNS 服务器的配置目录，用于配置 DNS 服务器（不是客户端）。

/etc/bind/ 目录说明
/etc/bind/ = BIND9 DNS 服务器配置目录
BIND = Berkeley Internet Name Domain
作用: 运行 DNS 服务器，提供域名解析服务

主要配置文件
1. 核心配置文件
/etc/bind/named.conf              # 主配置文件
/etc/bind/named.conf.options      # 选项配置
/etc/bind/named.conf.local        # 本地区域配置
/etc/bind/named.conf.default-zones # 默认区域
2. 区域文件（Zone Files）
/etc/bind/db.local                # 本地正向解析
/etc/bind/db.127                  # 本地反向解析
/etc/bind/db.root                 # 根服务器
/etc/bind/db.empty                # 空区域
/etc/bind/zones/                  # 自定义区域文件目录
查看 BIND 配置

# 查看主配置
cat /etc/bind/named.conf

# 查看选项配置
cat /etc/bind/named.conf.options

# 查看本地区域配置
cat /etc/bind/named.conf.local

# 查看所有配置文件
ls -la /etc/bind/

# 查看区域文件
cat /etc/bind/db.*
```

`DNS` 相关配置

```bash
# 查看 DNS 服务器配置
cat /etc/resolv.conf

# 查看 hosts 文件
cat /etc/hosts

# 查看 systemd-resolved 配置
systemd-resolve --status
resolvectl status
```

`DNS` 查询命令

```bash
# nslookup - 查询 DNS
nslookup venus.hmv

# dig - 详细 DNS 查询
dig venus.hmv
dig venus.hmv ANY
dig venus.hmv TXT
dig venus.hmv A
dig venus.hmv AAAA

# host - 简单 DNS 查询
host venus.hmv
host -a venus.hmv
```

```bash
leona@venus:~$ host venus.hmv
venus.hmv has address 172.66.0.10
leona@venus:~$ curl 172.66.0.10
33EtHoz9a0w2Yqo
```

这个密码是错误的。

```bash
cat /etc/bind/db.venus.hmv

;
; BIND data file for local loopback interface
;
    604800
@       IN      SOA     ns1.venus.hmv. root.venus.hmv. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

;@      IN      NS      localhost.
;@      IN      A       127.0.0.1
;@      IN      AAAA    ::1
@       IN      NS      ns1.venus.hmv.

;IP address of Name Server

ns1     IN      A       127.0.0.1
ava IN      TXT     oCXBeeEeYFX34NU
```

## level50

关卡说明：The password of maria is somewhere...

密码是：.--. .- .--. .- .--. .- .-. .- -.. .. ... .
