---
layout: article
title: WireGuard安装笔记
aside:
  toc: true
tags: wireguard tech
key: wireguard-install
comments: true
---

## 内核问题

首先按照[官网的手册](https://www.wireguard.com/install)找到自己的发行版安装。

由于wireguard已经并入内核，对于非标准或者太老内核可能需要更新，而ssh过去的机器重启之后是看不到grub的，因此需要在重启前先设置好grub使用新的内核。

1. 安装新的内核
2. 重构grub配置：`grub2-mkconfig`
3. 查看已有内核列表：`awk -F\' '$1=='menuentry " {print $2}' /etc/grub2.cfg`
4. 更改默认内核：`grub2-editenv - set saved_entry='{内核名称}'`

## 服务器端

生成公钥和私钥（所有端都需要）：

{% highlight shell %}
# cd /etc/wiregurad
# wg genkey | tee privatekey | wg publickey > publickey
{% endhighlight %}

注意配置文件里也有密钥信息，因此`/etc/wireguard`最好设置成600的权限。

随后在`/etc/wiregurad/`中创建`wg0.conf`：

{% highlight shell %}
[Interface]
Address = 10.0.0.1
ListenPort = [端口号]
PrivateKey = [服务器端的私钥] 
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
Post Down = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
DNS = 8.8.8.8

[Peer]
PublicKey = [客户端1的公钥]
AllowedIPs = 10.0.0.2/32

[Peer]
PublicKey = [客户端2的公钥]
AllowedIPs = 10.0.0.3/32
{% endhighlight %}

- IP地址根据自己的需要调整，不冲突即可；
- 密钥的代码直接复制，不需要加引号；
- 每个客户端都需要写一个Peer。

如果服务器端没有开启ipv4转发，需要在`/etc/sysctl.conf`中加一句`net.ipv4.ip_forward=1`，然后加载新设置：`# sysctl -p`

最后启动wireguard，如果配置文件有误，此处会报错：

{% highlight shell %}
# wg-quick up wg0
{% endhighlight %}

## 客户端

客户端同样在`/etc/wireguard/`下建立`wg0.conf`（windows的话在"我的文档"中建一个文件夹）：

{% highlight shell %}
[Interface]
Address = 10.0.0.2
PrivateKey = [客户端1的私钥]

[Peer]
PublicKey = [服务器端的公钥]
AllowedIPs = 10.0.0.0/24
Endpoint = [服务器端的公网IP]:[端口]
PersistentKeepalive = 25
{% endhighlight %}

服务器端与客户端均配置好之后，可以通过`wg-quick up wg0`和`wg-quick down wg0`启动与停止，也可以加入开机启动：

{% highlight shell %}
# # systemctl enable wg-quick@wg0 --now
{% endhighlight %}

## unraid

unraid的app中有wiregurad，不过网页端要求先配置服务器端，再添加peer。此时可以通过shell直接编辑`/etc/wireguard/`下的文件，启动之后再回到网页端就能看到刚才的配置，服务器端的配置不全无需理会。

## 参考资料

- [https://sh.alynx.one/posts/WireGuard-Usage/](https://sh.alynx.one/posts/WireGuard-Usage/)
- [https://eddyemma.com/blog/2018/08/21/%E5%8D%87%E7%BA%A7centos%E5%86%85%E6%A0%B8/](https://eddyemma.com/blog/2018/08/21/%E5%8D%87%E7%BA%A7centos%E5%86%85%E6%A0%B8/)
- [https://www.centos.bz/2017/08/upgrade-centos-7-6-kernel-to-4-12-4/](https://www.centos.bz/2017/08/upgrade-centos-7-6-kernel-to-4-12-4/)
