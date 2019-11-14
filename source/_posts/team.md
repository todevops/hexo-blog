---
title: Linux 聚合链路
date: 2018-12-03
categories:
  - linux
tags:
  - linux
  - network
---

<!-- more -->

broadcast（可将数据传送到所有端口）  
round-robin（可按顺序将数据传送到所有端口）  
active-backup（使用一个端口或链接时其他则处于备用状态）  
loadbalance（使用主动 Tx 负载平衡及基于 BPF 的 Tx 端口选择程序）  
lacp（采用 802.3ad 链接合并控制协议）  

## 1. 安装 teamd
默认不会安装网络成组守护进程 teamd。要安装 teamd
```bash
yum -y install teamd
```

## 2. 添加配置
```bash
# 使用名称 team-ServerA 创建新的成组接口
nmcli connection add type team ifname team-ServerA
nmcli con show team-ServerA
# 更改为成组分配的名称：
nmcli con mod old-team-name connection.id new-team-name
# 载入成组配置文件
nmcli connection modify team-name team.config JSON-config
# 检查 team.config 属性
nmcli con show team-name | grep team.config
# 在 Team0 中添加名为 Team0-port1 的接口 eth0
nmcli con add type team-slave con-name Team0-port1 ifname eth0 master Team0
# 添加另一个名为 Team0-port2 的接口 eth1
nmcli con add type team-slave con-name Team0-port2 ifname eth1 master Team0
# 激活端口
nmcli connection up Team0-port1
nmcli connection up Team0-port2
# 验证是否已激活成组接口
ip link
# 或者使用命令启用该接口组
nmcli connection up Team0
```
## 3. 配置文件
要创建网络成组，作为成组端口或链接接口的虚拟接口需要一个 JSON 格式的配置文件。快捷的方法是复制示例配置文件，然后使用有 root 授权的编辑器进行编辑。  
```bash
# 列出可用示例配置文件
ls /usr/share/doc/teamd-*/example_configs/
# 请使用以下命令查看包含的文件之一，比如 activebackup_ethtool_1.conf：
cat /usr/share/doc/teamd-*/example_configs/activebackup_ethtool_1.conf
```
