---
title: 群晖硬盘损毁抢救
tags: synology tech
key: synology-disk-error
comments: true
---

群晖对硬盘的检查很迷，而且提示损毁之后系统里直接无法查看，需要使用命令行挂载抢救。

<!--more-->

- `fdisk -l` 确定硬盘分区(假设是`/dev/hdd3`)
- `cat /proc/mdstat` 查看当前md的情况，如果分区已经挂上了，记住它的路径(假设是`/md3`)
- 如果上一步发现分区没有挂上，使用`mdadm -Cf /dev/md3 -e1.2 -n1 -l1 /dev/hdd3`挂上
- 使用fsck类命令检查
- `mount /dev/md3 /mnt/tmp`挂上分区，此时进入`/mnt/tmp`已经可以看到先前的文件了
- 修改`/etc/samba/smb.share.conf`，照着上面已有的共享目录配置抄一个，把路径改成`/mnt/tmp`，随后在windows的网络文件夹中就能看到这个目录，赶紧把数据复制出来吧
