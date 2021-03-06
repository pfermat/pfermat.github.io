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

<!--more-->

## frpc配置

{% highlight shell %}
[ovpn]
type = udp
local_port = 1194
remote_port = 1194
{% endhighlight %}

frpc的其他配置请参考手册自行添加；
此外有传言1194端口可能被封，可以考虑将其设置在其他端口上。

## openvpn配置

我是在虚拟机上跑的，选用了alpine linux的虚拟机版本，装完之后硬盘占用150M，内存占用100M，非常适合24小时运行的低功耗小鸡。

### 生成证书

首先去[easy-rsa 2](https://github.com/OpenVPN/easy-rsa-old)下载easy-rsa软件，随便找个地方解压。

找到`easy-rsa`的目录（有一堆可执行文件的那个），修改`vars`文件，将`KEY_COUNTRY`, `KEY_PROVINCE`, `KEY_CITY`, `KEY_ORG`, `KEY_EMAIL`都填上。（默认就有，保证非空即可）

初始化PKI，运行如下命令：

{% highlight shell %}
$ . ./vars
$ ./clean-all
$ ./build-ca
{% endhighlight %}

如果最后一步提示找不到`openssl.cnf`，翻一下目录，把类似`openssl-1.0.0.cnf`的文件改名即可。

在CA生成过程中，之前vars中填写的信息会自动带上，common name自己取一个。

随后为服务器和客户端生成证书文件：

{% highlight shell %}
$ ./build-key-server server
$ ./build-key client1
$ ./build-key client2
$ ./build-key client3
{% endhighlight %}

随后生成Diffie Hellman参数：

{% highlight shell %}
$ ./build-dh
{% endhighlight %}

进入`keys`目录，会发现一堆文件，其中`ca.crt`是服务器端与客户端均需要的，`client*`是对应客户端需要的，其他文件一般仅在服务器端需要。

### 配置服务器端

下载服务器端和客户端的样本[配置文件](https://github.com/OpenVPN/openvpn/tree/master/sample/sample-config-files)，根据上述配置，大致需要如下的参数：

{% highlight shell %}
port 1194
proto udp
dev tun
# 以下四行的文件名及路径按照实际情况修改
ca ca.crt
cert server.cert
key server.key
dh dh2048.pem
topology subnet
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
keepalive 10 120
chiper AES-256-CBC
max-clients 100
user nobody
group nobody
persist-key
persist-tun
# 日志文件位置
status /var/log/openvpn-status.log
log-append /var/log/openvpn.log
# 日志级别，默认是3，从0--9数字越多信息越多
verb 3
{% endhighlight %}

检查一下防火墙和TUN/TAP，随后启动服务器端。

### 配置客户端

客户端的配置与服务器端基本一致，对照着抄就可以。需要注意的是服务器地址，因为使用了frp转发，因此填写的应该是frps公网服务器的地址与frpc中`remote_port`的端口。

### 其他配置

**tls-auth**

生成一个key：

{% highlight shell %}
$ openvpn --genkey --secret ta.key
{% endhighlight %}

将ta.key文件分别放到服务器端和客户端，在服务器端的配置文件中增加：

{% highlight shell %}
$ tls-auth ta.key 0
{% endhighlight %}

在客户端的配置文件中增加：

{% highlight shell %}
$ tls-auth ta.key 1
{% endhighlight %}

**访问内网其他机器**

假设内网网段为192.168.0.0/24，按照默认设置，服务器端的ip是10.8.0.1，客户端分到的网段是10.8.0.1/24，虽然客户端可以访问服务器端，但是访问不了内网的其他机器，需要在服务器端设置。

第一步，在服务器端的配置文件中增加：

{% highlight shell %}
push "route 192.168.0.0 255.255.255.0"
{% endhighlight %}

第二步，设置服务器端的iptables规则：

{% highlight shell %}
# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
{% endhighlight %}

## 参考资料

- [https://openvpn.net/community-resources/how-to/](https://openvpn.net/community-resources/how-to/)
- [https://www.ilanni.com/?p=9877](https://www.ilanni.com/?p=9877)
