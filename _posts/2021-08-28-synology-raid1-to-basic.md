---
title: 群晖raid1改为basic
tags: synology shr raid1 basic
key: synology-raid1-to-basic
comments: true
---

将两盘raid1(SHR)改为basic。

<!--more-->

- 拔掉raid1中的一个硬盘；
- `cat /proc/mdstat`查看当前阵列状态；
- md0/md1一般是群晖的系统分区，从md2开始确认硬盘对应的分区；
- `mdadm --grow --raid-devices=1 --force /dev/md2`
- 在存储空间管理中确认已变为basic.
