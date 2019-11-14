---
title: Linux 网卡绑定
date: 2018-12-02
categories:
  - linux
tags:
  - linux
  - network
---

<!-- more -->

## 1. 常见的网卡绑定驱动模式:
+ mod=0 (balance-rr) Round-robin 衡抡循环策略
特点：  
传输数据包顺序是依次传输（即：第1个包走eth0，下一个包就走eth1.一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降  
+ mode=1 (active-backup) 主-备份策略
特点：  
只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond的MAC地址是唯一的，以避免switch(交换机)发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有 N 个网络接口的情况下，资源利用率为1/N  
+ mode=2 (balance-xor) XOR policy 平衡策略
特点：  
基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力  
+ mode=3 (broadcast) 广播策略  
特点：  
在每个slave接口上传输每个数据包，此模式提供了容错能力  
+ mode=4 (802.3ad) IEEE 802.3ad 动态链接聚合
特点：  
创建一个聚合组，它们共享同样的速率和双工设定。根据802.3ad规范将多个slave工作在同一个激活的聚合体下。
外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选项从缺省的XOR策略改变到其他策略。需要注意的 是，并不是所有的传输策略都是802.3ad适应的，尤其考虑到在802.3ad标准43.2.4章节提及的包乱序问题。不同的实现可能会有不同的适应性。  
必要条件：  
条件1：ethtool支持获取每个slave的速率和双工设定  
条件2：switch(交换机)支持IEEE 802.3ad Dynamic link aggregation  
条件3：大多数switch(交换机)需要经过特定配置才能支持802.3ad模式  
+ mode=5 (balance-tlb) Adaptive transmit load balancing 适配器传输负载均衡
特点：  
不需要任何特别的switch(交换机)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。  
必要条件：  
ethtool支持获取每个slave的速率  
+ mode=6 (balance-alb) Adaptive load balancing 适配器适应性负载均衡
特点：  
该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡(receive load balance, rlb)，而且不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

## 2. 使用 nmcli 工具配置网卡绑定
```bash
nmcli con add type bond con-name mybond0 ifname bond0 mode balance-alb
nmcli con add type bond-slave ifname ens37 master mybond0
nmcli con add type bond-slave ifname ens38 master mybond0
nmcli con up bond-slave-ens7
nmcli con modify mybond0 ipv4.method manual
nmcli con modify mybond0 ipv4.addresses 192.168.78.137/24
nmcli con modify mybond0 ipv4.gateway 192.168.78.2
nmcli con modify mybond0 ipv4.dns 192.168.78.2
```

## 3. 使用命令行界面配置网卡绑定
```bash
# 显示 boding 模块信息
modinfo bonding
```

### 3.1 创建频道绑定接口
在 /etc/sysconfig/network-scripts/ 目录中创建名为 ifcfg-bondN 的文件，使用接口号码替换 N，比如 0。
```
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=192.168.1.1
PREFIX=24
ONBOOT=yes
BOOTPROTO=none
BONDING_OPTS="bonding parameters separated by spaces"
```

### 3.2 创建从属接口
```
DEVICE=ethN
NAME=bond0-slave
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

### 3.3 激活频道绑定
```bash
ifup ifcfg-eth0
ifup ifcfg-eth1
# 生效更改
# nmcli con load /etc/sysconfig/network-script/ifcfg-device
nmcli con reload
# 查看网卡绑定接口状态
ip link show
```