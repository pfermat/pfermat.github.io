---
title: 群晖使用绿联usb3.0千兆网卡
tags: synology tech
key: synology-usb-net
comments: true
---

由于群晖的DS218+只有一个千兆网口，想直接从主板上加网卡是不太可能的，而后置的usb3.0接口闲着也是闲着，因此将暂时闲置的绿联usb3.0千兆网卡插上去试试。

<!--more-->

不过似乎从DSM6.2开始，群晖就把AX88179的驱动删了，所以需要自己编译驱动。
幸好已经有大神做了这件事：
[https://xpenology.com/forum/topic/28321-driver-extension-jun-103b104b-for-dsm623-for-918-3615xs-3617xs/](https://xpenology.com/forum/topic/28321-driver-extension-jun-103b104b-for-dsm623-for-918-3615xs-3617xs/) 从这里下载文件之后打开，找到如下三个模块：`mii`, `usbnet`, `ax88179_178a`.
将三个文件复制到`/lib/moduels/`目录下，依次运行以下命令：
{% highlight shell %}
# insmod /lib/modules/mii.ko
# insmod /lib/modules/usbnet.ko
# insmod /lib/modules/ax88179_178a.ko
# ifconfig eth1 up
{% endhighlight %}

如果一切正常，`eth1`起来之后，就能在DSM中看到第二个网卡了。
