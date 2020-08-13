---
layout: article
title: CentOS7下wordpress安装
aside:
  toc: true
tags: centos wordpress tech
key: centos7-install-wordpress
comments: true
---

题外话：digitalocean的文档很不错，很多地方我都是直接拿来用的。

<!--more-->

## LAMP安装

### apache(httpd)

在CentOS下搜apache的话会出来完全不同的东西，因为其实应该叫httpd……

- 安装
{% highlight shell %}
$ sudo yum install httpd
{% endhighlight %}

- 启动
{% highlight shell %}
$ sudo systemctl start httpd.service
{% endhighlight %}

- 设置成开机启动
{% highlight shell %}
$ sudo systemctl enalbe httpd.service
{% endhighlight %}

### mysql(mariadb)

- 安装、启动
{% highlight shell %}
$ sudo yum install mariadb-server mariadb
$ sudo systemctl start mariadb
{% endhighlight %}

- 安全设置脚本（除了设置root密码之外，可以一路回车）
{% highlight shell %}
$ sudo mysql_secure_installation
{% endhighlight %}

- 设置开机启动
{% highlight shell %}
$ sudo systemctl enable mariadb.service
{% endhighlight %}

### php

- 安装
{% highlight shell %}
$ sudo yum install php php-mysql
{% endhighlight %}

- 重启apache
{% highlight shell %}
$ sudo systemctl restart httpd.service
{% endhighlight %}

随后可以测试一下php是否正常运行，在CentOS默认的网站目录/var/www/html/下创建一个ino.php文件：
{% highlight php %}
<?php phpinfo(); ?>
{% endhighlight %}
随后用浏览器打开http://你的ip或域名/info.php，如果能看到页面就说明装好了。
**别忘了把这个文件删了。**

然而，等装完wordpress之后你会发现，它提示php版本过低，而官方库里的php并没有高版本，因此需要再安装一次php7。

- 启用remi repo
{% highlight shell %}
$ sudo yum install epel-release yum-utils
$ sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
{% endhighlight %}

- 安装php7.3和常用模块
{% highlight shell %}
$ sudo yum-config-manager --enable remi-php73
$ sudo yum install php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysqlnd
{% endhighlight %}

看一下php现在的版本：
{% highlight shell %}
$ php -v
{% endhighlight %}
最后重启apache

## wordpress安装

### 创建数据库

进入mysql
{% highlight shell %}
$ mysql -u root -p
{% endhighlight %}

在mysql的控制台依次输入如下命令（注意将其中的数据库名、用户名和密码换成自己的）：
{% highlight sql %}
> CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';
> FLUSH PRIVILEGES;
> exit
{% endhighlight %}
