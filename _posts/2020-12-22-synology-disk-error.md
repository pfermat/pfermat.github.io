---
title: How to solve Synology disk error
tags: synology tech
key: synology-disk-error
comments: true
---

Even small error status in SMART will cause synology report disk error. And refuse to operate on that disk. We need use command line to solve this.

<!--more-->

- `fdisk -l` Check partitions on the disk (eg. `/dev/hdd3`)
- `cat /proc/mdstat` Check the status of md (soft raid). If the partition is correctly mounted, remember its name (eg. `/md3`)
- If you cannot see the partition on the previous command, use `mdadm -Cf /dev/md3 -e1.2 -n1 -l1 /dev/hdd3` to mount it.
- Use `fsck` to check the disk.
- `mount /dev/md3 /mnt/tmp` Mount the partition on a temp directory. You can cd into it and check the contents.
- If you would like to copy the files under Windows, modify `/etc/samba/smb.share.conf`
