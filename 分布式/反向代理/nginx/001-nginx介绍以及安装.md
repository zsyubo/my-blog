# 简介

Nginx是一个免费、开源的、高性能的`HTTP服务器和反向代理`。兼具高并发和内存占用低的特点。

**优点**

支持高并发、内存消耗少、配置文件简单丰富、支持Rewrite重新规则、内置健康检查功能、节省带宽、稳定性搞。

## 正向代理

正向代理proxy和client一般同一个局域网，对server透明（也就是服务端不知道具体是哪一个客户端访问的），常见的如VPN

## 反向代理

反向代理proxy和server一般同属于一个局域网，对client，也就是客户端不知道具体访问的哪一个服务器。

<img src="https://s2.ax1x.com/2020/02/11/1TkvGR.png" style="zoom:70%;" />

# 安装

这里以Ubuntu作为实例，最简单的直接使用apt安装

```shell
# sudo apt install nginx
```

如果需要查看安装路径信息可以使用：

```shell
# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
```

`/etc/nginx`是默认配置路径。

通过命令`systemctl status nginx`可以查看nginx状态。

# 基础命令

**启动**

**重新载入配置文件**

`nginx -s reload`

**重启**

`nginx -s reopen`

**停止**

`nginx -s stop`