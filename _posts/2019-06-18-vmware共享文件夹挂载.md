---
title: vmware共享文件夹挂载
tags: vmware tech
key: vmware-mount-shared-folders
comments: true
---

{% highlight shell %}
$ vmhgfs-fuse .host:/$(vmware-hgfsclient) ~/some_mount_point
{% endhighlight %}

<!--more-->

[Enabling shared folders with open-vm-tools][source]

[source]:https://askubuntu.com/questions/580319/enabling-shared-folders-with-open-vm-tools
