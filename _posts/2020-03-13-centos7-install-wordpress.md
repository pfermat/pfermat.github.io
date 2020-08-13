---
layout: article
title: CentOS7下wordpress安装
aside:
  toc: true
tags: centos lamp wordpress tech
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

### 安装wordpress

- 下载和解压最新版wordpress：
{% highlight shell %}
$ wget http://wordpress.org/latest.tar.gz
$ tar xzvf latest.tar.gz
{% endhighlight %}
- 把文件复制到www目录：
{% highlight shell %}
$ cp -R ./wordpress /var/www/
{% endhighlight %}
- 创建一个上传文件的目录：
{% highlight shell %}
$ mkdir /var/www/wordpress/wp-content/uploads
{% endhighlight %}
- 改一下用户：
{% highlight shell %}
$ sudo chown -R apache:apache /var/www/wordpress/*
{% endhighlight %}

### 配置wordpress

{% highlight shell %}
$ cp wp-config-sample.php wp-config.php
{% endhighlight %}

打开wp-config.php，修改以下三处：

{% highlight shell %}
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');
{% endhighlight %}

## 通过acme.sh获取证书

### 安装acme.sh

{% highlight shell %}
$ curl https://get.acme.sh | sh
{% endhighlight %}

随后可以使用laias或者软链接到/usr/local/bin之类的方法，使acme.sh成为一个不用输入安装路径的命令。

### 生成证书

生成证书时需要确保域名解析正确，这里以cloudflare为例。至于cloudflare如何设置DNS之类的，可以自行搜索。
登录cloudflare后获取自己的API Key。
声明两个变量：

{% highlight shell %}
$ export CF_Token="sdfsdfsdfljlbjkljlkjsdfoiwje"
$ export CF_Account_ID="xxxxxxxxxxxxx"
{% endhighlight %}

随后开始获取证书，等待信息提示完成即可：

{% highlight shell %}
$ acme.sh --issue --dns dns_cf -d example.com -d www.example.com
{% endhighlight %}

### 安装证书

可以直接复制，不过acme推荐使用命令：

{% highlight shell %}
$ acme.sh --installcert -d example.com \
--cert-file /path/to/certfile/in/apache/cert.pem \
--key-file /path/to/keyfile/in/apache/key.pem \
--fullchain-file /path/to/fullchain/certfile/apache/fullchain.pem \
--reloadcmd "service apache2 reload"
{% endhighlight %}

## 配置apache

{% highlight shell %}
<VirtualHost *:443>
        ServerName your.domain

        ServerAdmin webmaster@example.com
        DocumentRoot /var/www/wordpress
        SSLEngine On

        SSLCertificateFile your/cert/file
        SSLCertificateKeyFile your/key/file

        Alias /wp-content /var/www/wordpress/wp-content
        <Directory /var/www/wordpress>
            Options FollowSymLinks
            AllowOverride Limit Options FileInfo
            DirectoryIndex index.php
            Require all granted
        </Directory>
        # 下面这段我也不知道为啥要加
        <Directory /var/www/wordpress/wp-content>
            Options FollowSymLinks
            Require all granted
        </Directory>
        # CentOS里这个变量是不对的，注意修改
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
{% endhighlight %}

最后打开浏览器，一切正常的话就可以看到wordpress的安装页面了。

## 参考链接

- [https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7#step-2-%E2%80%94-checking-your-web-server](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7#step-2-%E2%80%94-checking-your-web-server)
- [https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-centos-7#prerequisites](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-centos-7#prerequisites)
- [https://linuxize.com/post/install-php-7-on-centos-7/#configuring-php-7x-to-work-with-apache](https://linuxize.com/post/install-php-7-on-centos-7/#configuring-php-7x-to-work-with-apache)
- [https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
