---
title: 恢复firefox的session
tags: firefox tech
key: restore-firefox-session
comments: true
---

* 找到存储session的文件夹位置： C:\Documents and Settings\<Windows login/user name>\Application Data\Mozilla\Firefox\Profiles\<profile folder>

* 将sessionstore.jsonlz4改名

* 打开sessionstore-backups文件夹

* 将recovery.jsonlz4和recovery.backlz4改名

* 将需要的session文件复制到上级文件夹，并改名为sessionstore.jsonlz4

* 打开firefox
