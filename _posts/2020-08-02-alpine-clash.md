---
layout: article
title: Alpine Linux下配置clash
aside:
  toc: true
tags: alpine clash tech
key: alpine-clash-install
comments: true
---

之前使用operwrt作为旁路由，发现其实用不到那些路由器的功能，只需要一个透明代理即可，于是直接用alpine代替。

<!--more-->

## openrc启动脚本

{% highlight shell %}
#!/sbin/openrc-run
# 注意自行替换clash与配置文件的路径

depend() {
    need net
}

start() {
    ebegin "Starting clash"
    start-stop-daemon --background --start --exec /clash \
    --make-pidfile --pidfile /run/clash.pid \
    -- -d /etc/clash
    eend $?
}

stop() {
    ebegin "Stopping clash"
    start-stop-daemon --stop --exec /clash \
    --pidfile /run/clash.pid
    eend $?
}
{% endhighlight %}

## iptables规则

{% highlight shell %}
# SSH端口
iptables -t nat -A PREROUTING -p tcp --dport 22 -j ACCEPT
# clash规则 注意修改内网ip和转发端口
iptable -t nat -N CLASH
iptables -t nat -A CLASH -d 192.168.0.0/16 -j RETURN  
iptables -t nat -A CLASH -p tcp -j REDIRECT --to-ports 7892   
iptables -t nat -A PREROUTING -j CLASH  
# DNS
iptables -t nat -A PREROUTING -p udp -m udp --dport 53 -j DNAT --to-destination 192.168.0.1:53
{% endhighlight %}

## 参考资料

- [https://github.com/KOP-XIAO/Clash-Merlin/wiki](https://github.com/KOP-XIAO/Clash-Merlin/wiki)
- [http://big-elephants.com/2013-01/writing-your-own-init-scripts/](http://big-elephants.com/2013-01/writing-your-own-init-scripts/)
- [https://wiki.alpinelinux.org/wiki/Configure_Networking](https://wiki.alpinelinux.org/wiki/Configure_Networking)
- [https://github.com/Dreamacro/clash-dashboard/tree/gh-pages](https://github.com/Dreamacro/clash-dashboard/tree/gh-pages)
