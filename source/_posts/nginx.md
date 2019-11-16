---
title: Nginx
date: 2018-11-28
categories:
  - linux
tags:
  - linux
  - nginx
---

Nginx
<!-- more -->

## High-Performance Load Balancing（高可用负载均衡）
### 1.1 HTTP Load Balancing
+ 使用 NGINX 的 HTTP 模块使用 upstream 块在 HTTP 服务器上进行负载均衡

```conf
upstream backend {
    server 10.10.12.45:80       weight=1;
    server app.example.com:80   weight=2;
}
server {
    location / {
        proxy_pass http://backend;
    }
}
```

### 1.2 TCP Load Balancing
+ 使用 NGINX 的 stream 模块使用 upstream 块在 TCP 服务器上进行负载均衡

```conf
stream {
    upstream mysql_read {
        server read1.example.com:3306   weight=5;
        server read2.example.com:3306;
        server 10.10.12.34:3306         backup;
    }

    server {
        listen 3306;
        proxy_pass mysql_read;
    }
}
```

### 1.3 Load-Balancing 方法
+ 使用 NGINX 的负载平衡方法之一，例如least_conn, least_time, hash, ip_hash

这里将后端 upstream 池的负载平衡算法设置为最少连接。所有负载平衡算法（通用散列除外）都将是独立的指令，如前面的示例。通用散列采用单个参数（可以是变量的串联）来构建散列。

```conf
upstream backend {
    least_conn;
    server backend.example.com;
    server backend1.example.com;
}
```

并非所有请求或数据包都具有相同的权重。鉴于此，循环，甚至是先前示例中使用的加权循环，将不适合所有应用程序或交通流量的需要。NGINX提供了许多负载平衡算法，可用于适应特定的用例。 这些负载平衡算法或方法不仅可以选择，还可以配置。 以下负载平衡方法可用于 upstream HTTP，TCP 和 UDP 池：
1. Round robin（轮询）：  
默认负载平衡方法，按 upstream 池中服务器列表的顺序分配请求。对于加权循环，可以考虑权重，如果 upstream 服务器的容量变化，则可以使用加权循环。 权重的整数值越高，服务器在循环中的优势就越大。权重背后的算法只是加权平均的统计概率。循环法是默认的负载平衡算法，如果未指定其他算法，则使用该算法。
2. Least connections（）：  
另一种由 NGINX 提供的负载均衡方法。此方法通过将当前请求代理到具有通过 NGINX 代理的最少数量的打开连接的upstream服务器来平衡负载。在决定向哪个服务器发送连接时，最小连接（如循环）也会考虑权重。指令名称为 least_conn。
3. Least time（）：  
仅在 NGINX Plus 中可用，类似于最少连接，因为它代理具有最少数量的当前连接的upstream 服务器，但有利于具有最低平均响应时间的服务器。这种方法是最复杂的负载平衡算法之一，可满足高性能 Web 应用程序的需求。此算法是一个增加最少连接的值，因为少量连接并不一定意味着最快的响应。指令名称为 least_time。
4. Generic hash（）：  
管理员使用给定文本，请求或运行时的变量或两者来定义散列。NGINX 通过为当前请求生成哈希并将其放在 upstream 服务器上来分配服务器之间的负载。当您需要更多地控制发送请求的位置或确定哪些 upstream 服务器最有可能将数据缓存时，此方法非常有用。请注意，在池中添加或删除服务器时，将重新分配散列请求。该算法具有可选参数，一致，以最小化重新分布的影响。指令名称是 hash。
5. IP hash（）：  
仅支持HTTP，是最后一批。IP 哈希使用客户端 IP 地址作为哈希。与在通用散列中使用远程变量略有不同，此算法使用 IPv4 地址的前三个八位字节或整个 IPv6 地址。只要该服务器可用，此方法可确保客户端代理到同一个 upstream 服务器，这在会话状态受到关注且未由应用程序的共享内存处理时非常有用。 在分配散列时，此方法还会考虑权重参数。 指令名称为 ip_hash。

### 1.4 Connection Limiting
+ 使用 NGINX Plus 的 max_conns 参数限制 upstream 服务器的连接数

```conf
upstream backend {
    zone backends 64k;
    queue 750 timeout=30s;

    server webserver1.example.com max_conns=25;
    server webserver2.example.com max_conns=15;
}
```