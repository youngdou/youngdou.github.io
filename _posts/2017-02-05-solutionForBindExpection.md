---
layout: post
title:      "Linux下关闭开机自启动程序"
subtitle:   "关闭端口占用程序/关闭自启动程序的解决方法"

date:       2017-02-05
header-mask: 0.2
author:     "Young Dou"
tags:
    - Linux
---
# Linux下关闭开机自启动程序

## 问题
最近启动自己的项目的时候来时出现这个错误

```
caused by: java.net.BindException: 地址已在使用
```
我的项目用了8080端口，应该是8080端口被占用了


## 方案一

先找到占用这个端口的进程，然后kill掉就可以了

```bash
young@YOUNG:~$ sudo netstat -anp | grep 8080
[sudo] young 的密码：
tcp6 0 0 :::8080 :::* LISTEN 1423/java           
young@YOUNG:~$ sudo kill -9 1423
```

**但是这样的方法下，每次启动都要kill一次**，为了以绝后患，最好还是进行彻底的清查（也许还会清除出黑客的cracker也不一定呢）


## 方案二

```bash
young@YOUNG:/etc/lighttpd/conf-enabled$ sudo nmap localhost
[sudo] young 的密码：

Starting Nmap 7.12 ( https://nmap.org ) at 2017-02-05 21:07 CST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000060s latency).
Other addresses for localhost (not scanned): ::1
Not shown: 997 closed ports
PORT STATE SERVICE
631/tcp open ipp
3306/tcp open mysql
8080/tcp open http-proxy

Nmap done: 1 IP address (1 host up) scanned in 1.66 seconds
```

发现占用`8080`端口的是`http-proxy`（http代理）

这是啥呢？我们继续深入探究一下下

```bash
young@YOUNG:/etc/lighttpd/conf-enabled$ sudo lsof -i:8080
[sudo] young 的密码：
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
java 1423 tomcat7 54u IPv6 19148 0t0 TCP *:http-alt (LISTEN)
```

成功发现这玩意原来是我安装的`tomcat7`所自动的
我们知道，Linux中，开机自启动的程序启动脚本在 `/etc/init.d/`文件夹下的，我们寻根问底，一起过去look look什么情况

```bash
young@YOUNG:/etc/init.d$ ls /etc/init.d/ | grep tomcat
tomcat7
```

![img1](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2017-02-05/2017-02-05_212840.png) 

果然，有一个开机自启动的脚本。怎么办呢？直接删掉也许不太安全，所以我们可以使用一下的方法

下面关闭开机自启动

```bash
young@YOUNG:/etc/lighttpd/conf-enabled$ update-rc.d tomcat7 remove
Command 'update-rc.d' is available in '/usr/sbin/update-rc.d'
The command could not be located because '/usr/sbin' is not included in the PATH environment variable.
This is most likely caused by the lack of administrative priviledges associated with your user account.
update-rc.d: command not found
```

> 咦，权限不够诶，当然啦。
这里插一句：`update-rc.d`是系统管理员级别权限的命令，只能加`sudo`使用。不然万一被别有用心的一般用户就能使用来设置自启动的监控和后门程序，那主机可就沦为肉鸡了。

anyway,反正这样就可以了

```bash
young@YOUNG:/etc/lighttpd/conf-enabled$ sudo update-rc.d tomcat7 remove
```

真的可以了吗，我们继续查看`/etc/init.d/`下是否还有tomcat7脚本

```bash
young@YOUNG:/etc/init.d$ ls /etc/init.d/ | grep tomcat
```


果然没有了，还不信的话重启一下来看看呗。

```bash
young@YOUNG:~$ sudo netstat -anp | grep 8080
```

![img2](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2017-02-05/2017-02-05_212939.png) 

嘿嘿，果然啥都没有了。

以上。
