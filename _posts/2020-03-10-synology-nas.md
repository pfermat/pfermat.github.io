---
layout: article
title: 群晖NAS拾遗
aside:
  toc: true
tags: nas synology tech
key: synology-nas-notes
comments: true
---

## 安全

### 两步验证

1. 手机端安装Google Authenticator
2. 登陆DSM –> 右上角头像 –> 个人设置 –> 账号 –> 启用2步骤验证
3. 按照向导在手机端绑定
4. 重新登陆DSM，查看两步验证启用效果

### 公网安全

如果你的网络环境像我一样是电信大内网，那么可以省心公网暴露的问题（相对的，内网穿透成了另一个问题），大部分web端在外面根本不能直接访问；如果你有公网IP且暴露在外的服务比较多，那么请适当考虑以下几个问题：
（由于我没有公网IP，因此这里只提供建议）

- 打开防火墙，只选择需要用到的服务端口；
- 检查用户账号，普通账号只开放使用中的权限；
- 通过VPN连接；
- 另外注意检查路由器上的设置，不建议开放DMZ。

### 密码管理

现在撞库、明码存放、社工等层出不穷，使用密码管理器随机生成和存储密码成为必然事件。一般而言我们有两种选择：
1. 使用bitwarden，需要自己搭服务器；
2. 使用普通的密码管理器，通过云同步密码文件。
这里介绍第二种方法，使用KeePass。

- 使用Synology Drvie云同步密码文件，PC端当然没有问题，但是我的安卓端app无法找到DS Drive的文件，如果遇到同样的问题，可以换成比较旧的DS Cloud；
- 手机端选择顺手的app即可，我使用的是Keepass2Android，操作比较方便，有指纹设备的话还可以快速解锁。

## 内网穿透

如果你有公网IP，只是群晖的设备放在路由器之后，那么在路由器上设置好端口转发即可，DDNS之类的也是没问题的。
如果跟我一样是电信大内网，那么需要内网穿透，说白了就是在公网有一台机器作为转发的代理（又是一笔费用），具体配置方法可以参考我之前写的[frp简单配置](https://pfermat.github.io/2019/12/16/frp-conf.html)。
题外话：2019年下半年的时候，魔都这边曾经严查个人宽带的对外web服务，基本上查到有web页面的，轻则电话警告，重则断网签保证书。

## 挂机下载

### 群晖套件 Download Station

群晖提供了Download Station套件，无论是普通文件还是BT下载都没有问题，有手机端也有浏览器插件，如果你不愿意折腾或者不太懂这方面的技术，可以直接使用。

### aria2

aria2本体类似于一个命令行的守护进程，他会监听端口并进行下载任务，当然为了使用方便我们会使用带UI的客户端进行操作，推荐web端的aria2NG。

- 使用docker安装aria2本体
在docker中随便找一个排名靠前或者最近更新的aira2，启动的时候设置好端口和配置文件。配置时建议加上save-session参数，用于保存未完成的任务。
- PC端的客户端有两种选择，一种是直接在docker中找自带aria2NG的，启动之后直接通过web访问，另一种是使用独立的客户端，按照自己的喜好选择。
- 手机端如果使用了自带客户端的docker，可以打开浏览器直接访问，如果不喜欢浏览器，也可以找客户端，我使用的aria2app。
- 为了方便下载，可以在浏览器中装上aria2的插件，其实就是把下载地址发送给aira2监听的地址，firefox中我使用的是Send to Aria2 WE，Chrome也有类似的插件，可以自行查找。

## 网盘同步

群晖提供的Cloud Sync套件可以用来同步国内外常见的网盘

### Google Drive

1. 安装群晖的Cloud Sync套件
2. 添加Google云端硬盘，登陆google账号，选择本地文件夹，直至完成
3. 若一切正常，Cloud Sync应该已经开始同步google drive的文件，在总览中可以看到google drive中已使用的空间大小
4. 如果只是作为异地备份而不需要实时同步，可以在设置中将轮询期设置的大一点

### 度盘

虽然群晖的Cloud Sync提供了同步度盘的功能，但是不能选文件速度也很慢，加之度盘的不确定性，很不建议将其作为备份的选项。如果需要下度盘中的文件，可以使用下面docker中的方法。

## docker推荐

### transmission

由于需要下PT，而aria2这种流氓显示是要被叉出去的，可以选择使用transmission，另外PT和日常BT下载分开也是比较合理的。

docker直接用linuxserver的版本，除了常规目录之外，还可以把/usr/share/transmission/web和/root/.local/share/transmission/web映射到一个本地目录，随后将[transmission-web-control](https://github.com/ronggang/transmission-web-control)放进去。

### 度盘

拖一个johnshine/baidunetdisk-crossover-vnc，随后装一个vnc软件（也可以直接使用非vnc的版本）。因为我不是会员，具体加速情况未知，但是好处是可以24小时挂机，也不用开win虚拟机这种消耗比较大的东西。

### portainer

群晖自带的docker管理界面比较弱，可以在docker中装portainer来进行管理，用官方的即可。
另外如果网络中还有别的机器，portainer也可以接管它们的docker。

### syncthing

之前synology drive出了严重bug，导致我的windows系统无法正常使用，无奈只能卸载并寻找替代品。试了网盘中提及较多的nextcloud和seafile，都不符合我的要求，最后选定了稍显麻烦的syncthing。

syncthing更类似于分布式的同步软件，即便群晖在内网，它也会尝试点对点或者连接公网的中继点完成同步。

synology drive最新版本已经修复了2.0.1版本的bug，所以我又用了回来。从功能与便利性上而言，还是这家伙比较厉害。
