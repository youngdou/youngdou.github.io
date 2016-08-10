---
layout: post
title:      "ssd登录服务器保持进程运行"

date:       2016-08-10
header-mask: 0.2
author:     "Young Dou"
tags:
    - 云主机
    - Linux
    - Screen
---

## 阿里云部署中遇到的问题
- 在用ssd连接服务器以后运行了服务器脚本`python server.py`，但是一旦手动断开ssd和云服务器的连接，服务器脚本就会自动关闭导致无法访问

> **解决**：
> [参考传送门](http://www.ibm.com/developerworks/cn/linux/l-cn-screen/)
> 简单来说就是，用xshell登录云服务器ssh的时候是创建了一个会话，在断开ssh连接的时候会触发会话进程的signup信号，从而导致了进程被kill掉。
> 1. 可以通过`nohup python server.py &`来解决（其中`nohup`是使得相关进程忽略signup信号，`&`是让进程在后台执行）
> 2. 可以通过使用窗口管理软件`screen`来管理相关进程，从而避免被signup信号kill掉

> 具体讲解：
1. 首先在xshell登录ssh,并执行`top`:

![image](/img/post_imag/2016-08/101.png)

2. 在xshell开多一个会话窗口，执行`pstree`：

![image](/img/post_imag/2016-08/102.png)
++可见新窗口和旧窗口的父进程都来自sshd++

3. 在阿里云管理端开一个pstree:

![image](/img/post_imag/2016-08/103.png)

++发现阿里云开的bash的父进程是系统的login进程，而不是ssdh, 这也揭示了为什么在阿里云管理端运行的进程在断开连接时还可以在后台运行，而xshell连接（ssd连接）就不行*（参考开头的技术文章链接）*++

4. 使用刚才所说的方法进行解决

执行`nohup python server.py &` 然后关闭会话以后
在阿里云管理端可以看到：

![image](/img/post_imag/2016-08/104.png)

虽然sshd进程下已经没有了bash子进程，但是python进程还在，可见这个方法是可行的


> 方法二

1. 执行`screen vim test.md` 然后同时按下`ctrl+a+c`。这时会从vim界面回到bash界面；同样我们执行`screen python`,然后同时按下`ctrl+a+c`.
2. 此时，在xshell`pstree`看到如下：


![image](/img/post_imag/2016-08/105.png)

果然在screen的一个会话里面下开了多个窗口

3. 断开xshell连接，在管理端可见：
 

![image](/img/post_imag/2016-08/106.png)

嘿嘿嘿，诚不我欺，screen作为和sshd的兄弟进程出现了

4. 在管理端打开窗口：

`screen -ls `查看到一个实例，`screen -r [示例id]`（此处没有截到图）

> 注意:一定要搞清楚screen中的：screen, session,window三个概念，了解了这三个概念其实就了解了如何使用screen


> 参考资料：
> - [搞清楚screen, session,window三个概念](http://www.ibm.com/developerworks/cn/aix/library/au-gnu_screen/)
> - 清除一个session的方法: `screen -S [sessionName] -X quit`

screen -r [实例id]其实是切换到了xshell断开前的界面，没错断开前的历史界面，历史操作等都被screen保存了
![image](/img/post_imag/2016-08/107.png)

此时键入`pstree`:
![image](/img/post_imag/2016-08/108.png)
