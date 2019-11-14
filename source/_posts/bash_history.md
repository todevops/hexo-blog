---
title: history 添加时间戳
date: 2018-12-08
categories:
  - linux
tags:
  - linux
  - shell
---

<!-- more -->

## 添加环境变量
```
[root@client ~]# vim /etc/profile
```
```bash
HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S"
export HISTTIMEFORMAT
```