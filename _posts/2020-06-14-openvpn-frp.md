---
layout: article
title: openvpn与frp配置
aside:
  toc: true
tags: openvpn frp tech
key: openvpn-frp-conf
comments: true
---

## 材料

- 公网服务器一台，安装frps用来转发；
- 内网服务器一台，安装frpc，安装openvpn服务器端；
- 其他机器，安装openvpn客户端。

## frpc配置

```shell
[ovpn]
type = udp
local_port = 1194
remote_port = 1194
```

frpc的其他配置请参考手册自行添加；
此外有传言1194端口可能被封，可以考虑将其设置在其他端口上。
