---
title: "nibbles"
subtitle: ""
date: 2026-09-20T17:32:58+08:00
description: "htb的第一个靶机"
keywords: ""
tags: ["Box"]
categories: ["HTB Box"]
draft: true
comment: true
---

# HTB-Nibbles

## 前期准备工作

> 对于一个单个靶机，我觉得hackthebox的这套文件夹结构不是很适用，所以我就使用了自己的方式。

```sh
mkdir ./nibbles # 创建项目目录
cd ./nibbles
mkdir ./scan # 扫描的文件放 scan
mkdir ./logs # 存放日志文件
mkdir ./exploits # 存放漏洞利用脚本
mkdir ./loot # 存放战利品
mkdir ./screenshots # 存放关键步骤
```

> 日志记录工具，我选择的是tmux。

## 枚举
