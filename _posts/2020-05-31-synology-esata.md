---
title: 群晖修改内置接口为eSATA
tags: synology SATA tech
key: synology-to-esata
comments: true
---

群晖如果用内置接口的硬盘，是无法使用已有数据的，只能全部清空。而通过修改配置文件可以将内置SATA接口转成外置的eSATA接口，虽然个人不推荐这么做。

原理比较简单，就是修改`/etc.defaults/synoinfo.conf`文件中的`internalportcfg`与`esataportcfg`两个参数：

- `internalportcfg`的值从右往左，依次对应了每个接口是否为内置SATA接口（1为是，0为否）；
- `esataportcfg`的值从右往左，依次对应了每个接口是否为外置eSATA接口（1为是，0为否）。

以我的218+为例，原始的`internalportcfg=0x3=11(B)`，`esataportcfg=0x4=100(B)`；如果希望第二个SATA接口变成eSATA，则`internalportcfg=01(B)=0x1`，`esataportcfg=110(B)=0x6`.

保存后重启，改错参数系统有起不来的风险，谨慎。
