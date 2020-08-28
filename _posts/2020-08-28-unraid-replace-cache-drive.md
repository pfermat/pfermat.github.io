---
title: unraid更换缓存盘
tags: unraid tech
key: unraid-replace-cache
comments: true
---

unraid如何更换缓存盘（翻译自官方wiki）。

<!--more-->

1. 准备好新的缓存盘，可以使用[preclear](https://forums.unraid.net/topic/54648-preclear-plugin/)清一下；
2. 停止所有使用了缓存盘的插件、docker、虚拟机：
    - 进入**Settings -> Scheduler**，修改mover的日程，保证其在近几日不会运行；
    - 停止所有虚拟机，随后进入**Settings -> VM Manager**，设置**Enable VMs**为No；
    - 停止所有docker，随后进入**Settings -> Docker**，设置**Enable Docker**为No；
    - 如果安装了Community Applications Appdata Backup插件，确保它在期间不会运行。
3. 点击**Shares**，设置所有目录的**Use cache disk**为**Yes**，这将使mover将所有数据从缓存盘中移出；
4. 检查array中有足够的空间，随后在Main菜单中点击**Move Now**启用Mover；
5. 当Mover完成后，检查缓存盘是否为空。
6. 停止array，移除旧的缓存盘；
7. 安装新的缓存盘（硬件意义）；
8. 启用新的缓存盘；
9. 设置缓存盘的文件系统；
10. 启动array，格式化缓存盘；
11. 将所有希望只存放在缓存盘上的文件夹设置为**Use cache disk: Prefer**；
12. 在Main菜单中点击**Move Now**；
13. 当Mover完成后，检查缓存盘中的文件夹；
14. 将所有希望只存放在缓存盘上的文件夹设置为**Use cache disk: Only**；
15. 在**Settings**中重新启动之前停止的各个项目，例如docker、虚拟机、Mover日程；
16. 重新启动docker和虚拟机。


## 参考资料
- [https://wiki.unraid.net/Replace_A_Cache_Drive](https://wiki.unraid.net/Replace_A_Cache_Drive)
