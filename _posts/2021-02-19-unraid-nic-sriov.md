---
title: Unraid网卡开启SR-IOV
tags: unraid sriov
key: synology-nic-sriov
comments: true
---

给unraid装上了捡垃圾X520，之前搜到可以开启SR-IOV，但是资料不多，尝试了几个方法之后顺利开启。

<!--more-->

有人说BIOS中必须有相应选项才支持，也有人说只要有IOMMU和VT-d就可以，我的拒保嘉并没有单独的SR-IOV选项但是也成功了，可能与芯片组有关系。

- 通过`lspci -nn | grep Eth`找到网卡的PCI编号（比如我的是03:00.0）
- `cat /sys/bus/pci/devices/0000:03:00.0/sriov-totalvfs` 查看支持的vf数量
- `echo 63 > /sys/bus/pci/devices/0000:03.00.0/sriov_numvfs` 生成vf，把这条命令写入`/boot/config/go`使得开机启动

生成完毕后，进入Tools-System Devices，可以看到一堆网卡，随后如同直通显卡一样把设备号加入启动选项。
