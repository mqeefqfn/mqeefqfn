---
layout: post
title:  "Welcome to Jekyll!"
date:   2015-11-14 16:52:07
categories: jekyll update
tags: jekyll update
image: /images/pic02.jpg
---
##SSH连接 

免密登录
查看本地 ~/.ssh/ 目录是否有 id_rsa.pub，如果没有，在本地创建公钥

然后把本地公钥追(id_ras.pub)加到远程机器的 ~/.ssh/authorized_keys
```shell script
cat id_rsa.pub >> .ssh/authorized_keys 
```
保持连接
配置服务端心跳检测
让 server 每隔30秒向 client 发送一个 keep-alive 包来保持连接:

向 /etc/ssh/sshd_config

ClientAliveInterval 30
ClientAliveCountMax 3
第二行配置表示如果发送 keep-alive 3次，客户端依然没有反应，则服务端 sshd 断开连接。
别忘记重启ssh服务

配置客户端
在 /etc/ssh/ssh_config 或者 ~/.ssh/config
添加
ServerAliveInterval 30
ServerAliveCountMax 5

本地 ssh 每隔30s向 server 端 sshd 发送 keep-alive 包，如果发送 5 次，server 无回应断开连接。

共享SSH连接
如果需要在多个窗口中打开同一个服务器连接而不需要重复登陆，可以尝试添加 ~/.ssh/config，添加两行

ControlMaster auto
ControlPath ~/.ssh/connection-%r@%h:%p
配置之后，第二条连接共享第一次建立的连接，加快速度。

添加长连接配置
ControlPersist 4h
每次SSH连接建立之后，此条连接会被保持 4小时，退出服务器之后依然可以重用。

配置连接中转

ForwardAgent yes
当需要从一台服务器连接另外一个服务器，而在两台服务器中传输数据时，可以不用通过本地电脑中转，直接配置以上 ForwardAgent 即可。

最终， ~/.ssh/config 下的配置:

{% highlight shell %}
Host *
	ForwardAgent yes
	ServerAliveInterval 30
	ServerAliveCountMax 4
	TCPKeepAlive no
	ControlMaster auto
	ControlPath ~/.ssh/connection-%r@%h:%p
	ControlPersist 4h
	Compression yes
{% endhighlight %}
