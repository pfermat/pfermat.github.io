---
title: How to mount nfs on alpine linux
tags: nfs alpine linux
key: nfs-alpine
comments: true
---

How to set NFS on alpine linux.

<!--more-->

# Install nfs-utils
{% highlight shell %}
$ apk add nfs-utils
$ rc-update add nfsmount
$ rc-service nfsmount start
{% endhighlight %}

# Mount NFS
{% highlight shell %}
$ mount -t nfs ${NFS_SEVER}:${NFS_DIR} /mnt
{% endhighlight %}

# Mount NFS on boot
Add a line in `/etc/fstab`:
{% highlight shell %}
${NFS_SEVER}:${NFS_DIR} /mnt nfs \_netdev 0 0
{% endhighlight %}

# LXC

If you use LXC in pve, an error may occured like this:
{% highlight shell %}
* Starting NFS statd ...
 * start-stop-daemon: failed to start `/usr/sbin/rpc.statd'                                                                                                         [ !! ]
 * ERROR: rpc.statd failed to start
 * ERROR: cannot start nfsmount as rpc.statd would not start
{% endhighlight %}

See ref to solve this problem.

# REF

- [https://www.hiroom2.com/2017/08/22/alpinelinux-3-6-nfs-utils-client-en/](https://www.hiroom2.com/2017/08/22/alpinelinux-3-6-nfs-utils-client-en/)
- [https://superuser.com/questions/1619623/alpine-linux-3-12-0-nfsmount-failing-due-to-rpc-statd-not-starting](https://superuser.com/questions/1619623/alpine-linux-3-12-0-nfsmount-failing-due-to-rpc-statd-not-starting)
