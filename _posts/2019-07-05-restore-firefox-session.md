---
title: How to find and restore firefox sessions
tags: firefox tech
key: restore-firefox-session
comments: true
---

* Locate to the directory where firefox stores sessions: C:\Documents and Settings\<Windows login/user name>\Application Data\Mozilla\Firefox\Profiles\<profile folder>

* Rename sessionstore.jsonlz4

* Open sessionstore-backups

* Rename recovery.jsonlz4 and recovery.backlz4

* Copy the session files you need to the upper directory and rename it to sessionstore.jsonlz4

* Open firefox
