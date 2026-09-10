---
title: "ThinkPHP 的一些知识"
date: 2026-05-10T16:30:34+08:00
draft: false
description: "ThinkPHP 相关知识点整理"
categories: ["靶机复盘"]
tags: ["HackMyVM", "Medium"]
---
初学者的疑问：

1. 哪个URL能访问到哪个实际函数 (**`ThinkPHP` 如何把请求解析成 应用/控制器/方法**)

`thinkphp` 有两种入口方式 :

* 显示路由
  * 通过`Route::rule()、Route::get()`手动写入匹配规则，`thinkphp`内部会按你写的规则匹配。
* 默认路由
  * 没写显式路由，或当前请求的URL没有匹配上你写的显示路由规则，并且允许默认路由，框架会按照 "应用 / 控制器 / 方法" 去找函数。

引出第二个问题：

2. 如何区分 应用 / 控制器 / 方法

```
/应用/控制器/方法
```

对应到实际的路径应该是

```
app/应用/controller/控制器.php 里的 方法()
```

例如：

```
/index/token/token
```

对应:

```
app/index/controller/Token.php
```

里的：

```
public function token()
```

**应用**：能作为` app/<name>/controller/...`，这样的独立业务分区存在；对应`url`里面的第一段。

**控制器**：负责接收请求并处理业务，`app/index/controller/<name>`；对应`url`里面的第二段。

**方法**：控制器类里的函数，真正执行业务逻辑；对应`url`里的第三段。

**默认路由**

从代码找 URL

```
app/应用/controller/控制器.php 里的 public function 方法
↓
默认可能访问为
/应用/控制器/方法
```

从 URL 找代码

```
/应用/控制器/方法
↓
app/应用/controller/控制器.php
↓
public function 方法
```

**显示路由**

```
Route::rule("sb/:a/:b","Admin/hello");
↓
/应用/sb/1/2 # 这会映射到Admin/hello
```
