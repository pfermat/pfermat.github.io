---
title: git提交空目录
tags: git tech
key: git-commit-empty-folder
comments: true
---

{% highlight shell %}
$ find . -type d -empty -exec touch {}/.gitignore \;
{% endhighlight %}
