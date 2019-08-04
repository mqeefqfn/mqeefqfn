---
layout: post
title:  "怎么样为git设置多个key"
date:   2019-08-03 12:00:00
categories: linux git
tags: git
image: /images/pic01.jpg
---
为git设置不同的key其实很简单，git使用ssh作为通信手段，所以只需要设置为不同的host设置不同的key即可
方法是在 ~/.ssh/config 为不同的host添加不同的key

{% highlight bash %}
#gitlab
Host gitlab.com
  HostName gitlab.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_gitlab

#github
Host github.com
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_github
{% endhighlight %}
### refrence
[如何为git设置多个key](http://einverne.github.io/post/2015/08/git-with-multi-ssh-key.html)
