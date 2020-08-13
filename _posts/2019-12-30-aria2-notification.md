---
layout: article
title: aria2下载完成后发通知
aside:
  toc: true
tags: aria2 tech
key: aria2-noti
comments: true
---

## 20200107更新

只是给telegram发消息的话，直接在shell里调API就行了：

{% highlight shell %}
curl -s "https://api.telegram.org/<BOT_TOKEN>/sendMessage?chat_id=<CHAT_ID>&text=xxx"
{% endhighlight %}


一句话总结：aria2下载完成 –> on-download-complete –> 发邮件 –> smtp to telegram –> 手机收到通知

主要是通知手机，因为人在家里的PC前面的话，不需要通知。

一个非常简单的办法，用Aria2App，挂上自己的RPC地址，直接就是手机端的管理界面了。

如果想要自力更生呢？比如我希望发邮件通知，然后发现群晖配好了邮件服务器也没用，因为电信把端口全封了；在小众软件上正好看到有个SMTP to telegram的项目，可以劫持smtp把邮件转到telegram的bot上，于是决定就这么办。
因为技术水平太差，中间遇到了不少坑，google了好久终于解决了，虽然有些方法是歪门邪道……

## telegram bot

在[https://github.com/KostyaEsmukov/smtp_to_telegram](https://github.com/KostyaEsmukov/smtp_to_telegram)中已经写了如何申请并启用telegram bot的方法，总结而言就是要拿到chat_id和bot_token这两个值。

## 设置smtp to telegram

docker直接搜smtp_to_telegram就可以找到上面那哥们写的东西，把chat_ids和bot_token两个变量配置好，端口映射2525（默认），启动容器。
在群晖的 控制面板 –> 通知设置 –> 电子邮件 中，可以勾选启用电子邮件通知：SMTP服务器填127.0.0.1，SMTP端口填2525，其他随意；最后点击<发送测试电子邮件>，顺利的话telegram会马上收到bot的通知。

## mailx发送邮件

其实用sendmail应该更方便，而且群晖装上邮件套件后自带sendmail，不过我看了会儿man发现完全不会用，于是转投mailx……
直接搜mailx的话会发现一些无关的内容，仔细找了找发现全名应该是”heirloom-mailx”，原名nail。docker上也有，直接拖下来启动。
启动之后进入容器的命令行，测试一下能否发送邮件：

{% highlight shell %}
mailx -S smtp=127.0.0.1:2525 -s "Test subject" -v youremail'
{% endhighlight %}

## 串联

如果是在普通linux服务器上运行，事情已经基本完成了：设置on-download-complete=mail.sh，在mail.sh中写发邮件命令就万事大吉了。
不过，由于docker中的容器是隔离的，怎么调用就成了问题。docker的文档读完天都黑了，本着能google就google反正我的nas在内网没啥安全问题，找了点特殊技巧解决，在意安全和原则的话可以再讨论合理的做法。

先放结论，脚本中最重要的那句发邮件命令这样写：

{% highlight shell %}
docker exec -e "FILENAME=$3" mailx /bin/sh -c 'echo $FILENAME | mailx -S smtp=127.0.0.1:2525 -s "Your subject" -v youremail'
{% endhighlight %}

- 如何在aria2的容器中运行mailx
用docker exec可以运行容器中的命令，但是容器中是没有docker命令的，为此参考[https://blog.csdn.net/catoop/article/details/91042007](https://blog.csdn.net/catoop/article/details/91042007)，将docker和docker.sock挂载到容器中，就可以成功地在aria2容器中运行mailx容器的命令了。

- 如何向mailx容器传递环境变量
aria2在下载完成后会提供三个变量，其中$3是文件路径，而通过-e "KEY=VALUE"可以将新的环境变量传递给运行中的容器。

综上，在aria2容器中运行mailx，并把文件路径传递过去，就可以完整地发送邮件了。
