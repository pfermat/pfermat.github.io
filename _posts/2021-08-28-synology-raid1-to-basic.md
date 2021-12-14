---
title: How to change raid1 to basic in synology
tags: synology shr raid1 basic
key: synology-raid1-to-basic
comments: true
---

Change raid1/SHR (two disks only) to basic.
Use as your own risk.

<!--more-->

- Eject one of the disk in raid1.
- `cat /proc/mdstat` to check the raid status.
- `md0` and `md1` are usually synology's system partitions. So user partitions begin at `md2`
- `mdadm --grow --raid-devices=1 --force /dev/md2` Force `md2` change to a one disk raid i.e. basic.
- Check the result in DSM.
