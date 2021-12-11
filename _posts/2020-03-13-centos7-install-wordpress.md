---
layout: article
title: Install wordpress under CentOS7
aside:
  toc: true
tags: centos lamp wordpress tech
key: centos7-install-wordpress
comments: true
---

Documents on digitalocean are well-written.

<!--more-->

## Install LAMP

### apache(httpd)

The package name of apache under CentOS is httpd, not 'apache' itself.

- Install
{% highlight shell %}
$ sudo yum install httpd
{% endhighlight %}

- Start
{% highlight shell %}
$ sudo systemctl start httpd.service
{% endhighlight %}

- Set on boot
{% highlight shell %}
$ sudo systemctl enalbe httpd.service
{% endhighlight %}

### mysql(mariadb)

- Install and start
{% highlight shell %}
$ sudo yum install mariadb-server mariadb
$ sudo systemctl start mariadb
{% endhighlight %}

- Security setting script (don't forget to set the root password)
{% highlight shell %}
$ sudo mysql_secure_installation
{% endhighlight %}

- Set on boot
{% highlight shell %}
$ sudo systemctl enable mariadb.service
{% endhighlight %}

### php

- Install
{% highlight shell %}
$ sudo yum install php php-mysql
{% endhighlight %}

- Restart apache
{% highlight shell %}
$ sudo systemctl restart httpd.service
{% endhighlight %}

Then we shall test while php is running correctly. Create an info.php under default web directory `/var/www/html/`.
{% highlight php %}
<?php phpinfo(); ?>
{% endhighlight %}
Use the browser to open http://[your ip or domain]/info.php, check the page.
**Delete info.php after everything is correct.**

After installing wordpress you may found it said php's version too low. But there is no higher version of php in repo, so we need to reinstall php7.

- Enable remi repo
{% highlight shell %}
$ sudo yum install epel-release yum-utils
$ sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
{% endhighlight %}

- Install php7.3 and useful modules
{% highlight shell %}
$ sudo yum-config-manager --enable remi-php73
$ sudo yum install php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysqlnd
{% endhighlight %}

Check php version
{% highlight shell %}
$ php -v
{% endhighlight %}
Then restart apache.

## Install wordpress

### Create database

Enter mysql console
{% highlight shell %}
$ mysql -u root -p
{% endhighlight %}

Enter the follow commands in console (change the database name, username and password)
{% highlight sql %}
> CREATE USER wordpressuser@localhost IDENTIFIED BY 'password';
> GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';
> FLUSH PRIVILEGES;
> exit
{% endhighlight %}

### Install wordpress

- Get the latest wordpress：
{% highlight shell %}
$ wget http://wordpress.org/latest.tar.gz
$ tar xzvf latest.tar.gz
{% endhighlight %}
- Copy files to `www` directory:
{% highlight shell %}
$ cp -R ./wordpress /var/www/
{% endhighlight %}
- Create a directory for uploading:
{% highlight shell %}
$ mkdir /var/www/wordpress/wp-content/uploads
{% endhighlight %}
- Change file owner:
{% highlight shell %}
$ sudo chown -R apache:apache /var/www/wordpress/*
{% endhighlight %}

### Configure wordpress

{% highlight shell %}
$ cp wp-config-sample.php wp-config.php
{% endhighlight %}

Edit these lines in `wp-config.php` (three lines):

{% highlight shell %}
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');
{% endhighlight %}

## Use acme.sh to get cert

### Install acme.sh

{% highlight shell %}
$ curl https://get.acme.sh | sh
{% endhighlight %}

You may link acme.sh to `/usr/local/bin`

### Generate cert

Make sure your domain is correctly resolved. Suppose we use cloudflare as nameserver.
Login to cloudflare and get your own API key.

Export two variabels:

{% highlight shell %}
$ export CF_Token="sdfsdfsdfljlbjkljlkjsdfoiwje"
$ export CF_Account_ID="xxxxxxxxxxxxx"
{% endhighlight %}

Get the cert:

{% highlight shell %}
$ acme.sh --issue --dns dns_cf -d example.com -d www.example.com
{% endhighlight %}

### Install cert

Use the command recommanded by acme:

{% highlight shell %}
$ acme.sh --installcert -d example.com \
--cert-file /path/to/certfile/in/apache/cert.pem \
--key-file /path/to/keyfile/in/apache/key.pem \
--fullchain-file /path/to/fullchain/certfile/apache/fullchain.pem \
--reloadcmd "service apache2 reload"
{% endhighlight %}

## Configure apache

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
        
        <Directory /var/www/wordpress/wp-content>
            Options FollowSymLinks
            Require all granted
        </Directory>
        # The variables here are not right in CentOS
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
{% endhighlight %}

Open the browser and following wordpress's installing steps.

## REF

- [https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7#step-2-%E2%80%94-checking-your-web-server](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-centos-7#step-2-%E2%80%94-checking-your-web-server)
- [https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-centos-7#prerequisites](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-centos-7#prerequisites)
- [https://linuxize.com/post/install-php-7-on-centos-7/#configuring-php-7x-to-work-with-apache](https://linuxize.com/post/install-php-7-on-centos-7/#configuring-php-7x-to-work-with-apache)
- [https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
