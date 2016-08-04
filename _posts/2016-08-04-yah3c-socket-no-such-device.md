---
layout: post
title:      "Yah3c 在deepin下运行的问题"
subtitle:   "deepin下安装yah3c出错(no such device)"
date:       2016-08-03
header-mask: 0.2
author:     "Young Dou"
tags:
    - deepin
    - Linux
    - yah3c
---

# deepin下安装yah3c出错

> 标题虽说是在deepin下安装出错，但是也许也适用于安装多linux系统的机子

---

## 问题

因为喜欢折腾，我的机子除了安装了`win10`以外还安装了`ubuntu`和`deepin`，但是在使用deepin安装yah3c连接校园网的时候出现了`socket.error: [Errno 19] No such device`的报错（具体报错如下）

```
youngdou@youngdou-pc:/etc$ sudo yah3c
0 - add a new user
1 - douyy3(eth0)
Your choice: 1
Traceback (most recent call last):
  File "/usr/local/bin/yah3c", line 5, in <module>
    yah3c.yah3c.main()
  File "/usr/local/lib/python2.7/dist-packages/yah3c/yah3c.py", line 126, in main
    start_yah3c(login_info)
  File "/usr/local/lib/python2.7/dist-packages/yah3c/yah3c.py", line 102, in start_yah3c
    yah3c = eapauth.EAPAuth(login_info)
  File "/usr/local/lib/python2.7/dist-packages/yah3c/eapauth.py", line 34, in __init__
    self.client.bind((login_info['ethernet_interface'], ETHERTYPE_PAE))
  File "/usr/lib/python2.7/socket.py", line 228, in meth
    return getattr(self._sock,name)(*args)
socket.error: [Errno 19] No such device

```
---

## 原因


- 由python的报错信息可知，代码错误的主要原因是脚本没有找到所设置的网卡`socket.error: [Errno 19] No such device`

- 这里的原因是创建账户的时候使用了默认设置网卡(eth0网卡)而我的机子的eth0网卡被别处占用了,导致了在deepin下找不到这个网卡`在ubuntu是可以设置为eth0的`

---

## 解决方案

既然是设置的网卡出了问题，那就给他设置一个可用的网卡就行了


1 . 首先查看机子的网卡信息
> sudo /sbin/iwconfig 

```
youngdou@youngdou-pc:/$ sudo /sbin/iwconfig 
[sudo] youngdou 的密码：
wlp4s0    IEEE 802.11bgn  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=15 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
          
enp5s0f2  no wireless extensions.

lo        no wireless extensions.

```
这里可以看到我的机子的网卡为`enp5s0f2`

---

2 .重新给账户设置网卡

根据yah3c中`readme.md`找到账户配置文件重新配置即可

> sudo vi /etc/yah3c.conf 

记得加上sudo获得权限
然后把`ethernet_interface=eth0`改为刚刚查到的网卡`enp5s0f2`，改完后如下

```
...
ethernet_interface = enp5s0f2
...

```
保存以后再用`sudo yah3c`登陆你的账号吧～是否大功告成了呢？

