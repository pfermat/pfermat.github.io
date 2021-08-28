---
title: 群晖中使用ramdisk
tags: synology ramdisk
key: synology-ramdisk
comments: true
---

在群晖中划出一部分内存挂载为ramdisk，并使用rsync同步文件。

<!--more-->

- 创建ramdisk
{% highlight shell %}
#!/bin/sh
# mount-ramdisk.sh

mount -t tmpfs -o size=1G ramdisk /mnt/ramdisk
sleep 1
rsync -ahW --no-compress /mnt/local/ /mnt/ramdisk
{% endhighlight %}

- 同步文件
{% highlight shell %}
#!/bin/sh
# sync-ramdisk.sh
rsync -ahW --no-compress --delete /mnt/ramdisk/ /mnt/local
{% endhighlight %}

## 参考资料
- [https://marc.tv/synology-ramdisk-anlegen/](https://marc.tv/synology-ramdisk-anlegen/)
