---
title: uWSGI 部署 Django 应用
date: 2019-01-07
categories:
  - linux
tags:
  - linux
  - python
---

<!-- more -->

+ demo项目的目录结构

```bash
demo/
├── db.sqlite3
├── demo
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── settings.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── wsgi.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── manage.py
└── requirements.txt
```

## 安装相关工具和依赖

```bash
[root@uwsgi ~]# yum -y install git wget httpd vim # 安装相关工具
[root@uwsgi ~]# yum -y install gcc zlib* openssl-devel # 安装编译工具和依赖库
```
## 安装 Python 环境
rhel系列无法通过yum直接安装python3，需要源码编译安装
```bash
[root@uwsgi ~]# pwd # 查看当前目录
/root
[root@uwsgi ~]# wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz # 下载python3.6.5
[root@uwsgi ~]# tar -zxvf Python-3.6.5.tgz # 解压
[root@uwsgi ~]# cd Python-3.6.5

[root@uwsgi Python-3.6.5]# pwd # 查看当前目录
/root/Python-3.6.5
[root@uwsgi Python-3.6.5]# ./configure # 编译
[root@uwsgi Python-3.6.5]# make && make install # 安装
[root@uwsgi Python-3.6.5]# which python3 # 查看python3的路径
/usr/local/bin/python3
```

## 安装 uWSGI
```bash
[root@uwsgi ~]# python3 -m pip install uwsgi
```
## 安装配置虚拟环境
```bash
[root@uwsgi ~]# python3 -m pip install virtualenv # 安装虚拟环境
[root@uwsgi ~]# virtualenv -p /usr/local/bin/python3 /venv # 配置虚拟环境
[root@uwsgi ~]# source /venv/bin/activate # 激活虚拟环境
(venv) [root@uwsgi ~]# pip install -r /root/demo/requirements.txt # 安装项目依赖
(venv) [root@uwsgi ~]# pip freeze # 查看依赖库是否安装
(venv) [root@uwsgi ~]# deactivate # 取消激活虚拟环境
[root@uwsgi ~]# 
```

## 修改 ALLOW_HOSTS
```bash
[root@uwsgi ~]# vim demo/demo/settings.py # 编辑项目中的 settings.py
```
```py
ALLOWED_HOSTS = ["192.168.41.130",]
```

## 配置 uWSGI
```bash
[root@uwsgi ~]# mkdir -p /var/log/uwsgi # 创建 uwsgi 的日志目录
[root@uwsgi ~]# vim /etc/uwsgi.ini # 创建 uwsgi.ini 配置文件
```

```conf
[uwsgi]
chdir=/root/demo
http=192.168.41.130:8000
home=/venv
module=demo.wsgi:application
master=True
pidfile=/tmp/demo.pid
max-requests=5000
daemonize=/var/log/uwsgi/demo.log
env=LANG=en_US.UTF-8
buffer-size=32768
```

## 启动 uWSGI
```bash
[root@uwsgi ~]# uwsgi --init /etc/uwsgi.ini # 以 /etc/uwsgi.ini 配置启动 uwsgi
```

## 注意事项

|路径|说明|
|-|-|
|/root/demo|项目路径|
|/usr/local/bin/python3|python3默认安装路径|
|/etc/uwsgi.ini|uwsgi配置文件|
|/var/log/uwsgi/demo.log|uwsgi日志文件|
|/venv|python虚拟环境路径|

未安装zlib*：
```
zipimport.ZipImportError: can't decompress data; zlib not available # 编译时报错
```
未安装openssl、openssl-devel：
```
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available # pip 安装 uwsgi 时报错
```
uwsgi 无法访问，日志报错：
```
invalid request block size: 21573 (max 4096)...skip # 在 /etc/uwsgi.ini 中添加 buffer-size=32768
```