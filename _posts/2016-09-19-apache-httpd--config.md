---
layout: post
title:      "apache httpd/apache2配置解读"
subtitle:   "apache httpd/apache配置文件的一些解读，主要参考"
date:       2016-09-19
header-mask: 0.2
author:     "Young Dou"
tags:
    - apache2
    - java
    - web后端
---

# apache httpd/apache2配置文件解读

> 首先说明一下，apache httpd和apache2其实是一个东西，[可以参考](http://blog.csdn.net/z507263441/article/details/39991229)

> 还需要说明的是apache2.4和之前的apache2.2还是有点区别的。[例如](http://www.tuicool.com/articles/IrMvYbe)所以在配置最新的apache2.4的时候十分要注意，谨慎参考网上的配置教程(这都是泪的教训好嘛)

## 一、配置文件目录结构

- 文件的路径一般是在/etc/apache2底下

![文件目录](/img/post_imag/2016-09/201609-1.png)

/etc/apache2/

- 文件结构：

```
|-- apache2.conf
|	`--  ports.conf
|-- mods-enabled
|	|-- *.load
|	`-- *.conf
|-- conf-enabled
|	`-- *.conf
`-- sites-enabled
 	`-- *.conf
```

### 1. 主配置文件apache2.conf

apache2.conf是apache2.conf是服务器的主要配置文件，其他模块的配置文件其实是通过在apache2.conf使用`Include`包含进apache2.conf中去的。在服务器启动的时候首先读取的是apache2.conf

> 需要说明的是有的版本安装后会有httpd.conf文件，其实那个是用于给用户进行配置的文件，也是通过`Include`包含进apache2.conf中去的。所以，如果没有发现httpd.conf也不要难过，振作起来......总之没有httpd.conf也没关系啦。

### 2. ports.conf

用于设置服务器的监听端口

> 该文件必须导入apache2.conf中

### 3. *-enable文件夹

#### Summary
首先，以下三个文件夹都是用来管理配置文件的

- mods-enabled/
- conf-enabled/ 
- sites-enabled/

准确说是`文件片段`，因为每个类型的配置都会归为一个配置文件(后缀为`.conf`，或者是`.load`),其中`*.conf`指的是具体的配置数据，`*.load`指的是该类型配置所需导入的内容,例如：

> mpm_even.load和mpm_even.conf都是配置线程内容的

- mpm_even.conf配置具体数据
![具体数据.conf](/img/post_imag/2016-09/201609-2.png)

- mpm_even.load导入相关内容，这里是导入模块
![导入.load](/img/post_imag/2016-09/201609-3.png)

#### Detail

- mods-enabled/
> 里面的配置是用于管理模块Modules的

- conf-enabled/ 
> 里面的配置是用来管理全局配置的

- sites-enabled/
> 里面的配置用于管理虚拟主机的配置的

### 4. *-available文件夹

#### Summary
*-availabl放的也是配置文件，但是这个是预留的还没有开启，激活的配置文件，要想激活相关的配置，就要把该配置文件symlink到对应的*-enable文件夹里面去，例如配置cgi服务，就要将mods-available文件夹的cgid.load和cgid.conf文件symlink到mods-enable文件夹当中去。

#### Detail

> 有必要强调一下，其实*-enable里面的配置要想激活，就要通过在*-available文件夹对其进行软链接symlink来实现。

- 所以，要修改配置的话，首先要在相关的-available文件夹里面修改，然后symlink到-enable文件夹。

- 好，那么问题来了，怎么进行symlink呢？

其实在安装apache2的时候就已经安装了模块管理工具，使用命令`a2enmod`启动模块（实际山就是实现symlink）,使用命令`a2dismod`关闭模块（取消symlink）

- 举个例子


假如我要激活mods-available中的file_cache配置文件

*激活前*
![激活前](/img/post_imag/2016-09/201609-5.png)

那么就使用命令

```shell
sudo a2dismod file_cache
```
然后服务器会自动重启
![激活配置](/img/post_imag/2016-09/201609-4.png)

*激活后*
![激活后](/img/post_imag/2016-09/201609-6.png)

同理使用`a2dismod`取消symlink



## 二、apache2.conf主配置文件的配置内容

> 此处主要介绍几个常用的配置内容，还有部分的配置没有写入本文

- Timeout 超时设置
- KeepAlive 是否允许一个连接多个请求
- MaxKeepAliveRequests 最大请求等待时间
- HostnameLookups 记录客户端名称还是IP地址
- ErrorLog ${APACHE_LOG_DIR}/error.log 服务器错误日志的保存路径

> `APACHE_LOG_DIR`是环境变量，要在/etc/apache2/envvars设置

- 引入了配置文件

```
# Include module configuration:
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf

# Include list of ports to listen on
Include ports.conf

# Include generic snippets of statements
IncludeOptional conf-enabled/*.conf

# Include the virtual host configurations:
IncludeOptional sites-enabled/*.conf

## 详情可以回顾本文章前面的内容
```