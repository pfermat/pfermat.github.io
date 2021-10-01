---
title: 在pixel4中启用Motion Sense
tags: pixel motion-sense
key: pixel4-motion-sense
comments: true
---

在不支持的地区启用Motion Sense。

<!--more-->

- 在Magisk中安装MagiskHide Props Config模块；
- 安装任意命令行app，如Termux；
- 运行命令行，输入su切到root；
- 输入props打开MagiskHide Props Config工具；
- 选择add/edit custom props；
- 选择new；
- 变量名输入`pixel.oslo.allowed_override`；
- 变量值输入`1`；
- 启动顺序选择post-fs-data；
- 保存，重启

## 参考资料
- [这个帖子的第95楼](https://forum.xda-developers.com/t/magisk-tasker-release-motion-sense-soli-oslo-mod.3993877/page-5)
