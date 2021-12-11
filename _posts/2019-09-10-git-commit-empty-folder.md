---
title: How to commit an empty folder in git
tags: git tech
key: git-commit-empty-folder
comments: true
---

{% highlight shell %}
$ find . -type d -empty -exec touch {}/.gitignore \;
{% endhighlight %}
