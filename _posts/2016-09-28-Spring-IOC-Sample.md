---
layout: post
title:      "Spring IOC原理与样例(使用maven)"
subtitle:   "IOC原理讲解，并且使用maven创建一个简单sample"
date:       2016-09-28
header-mask: 0.2
author:     "Young Dou"
tags:
    - java
    - web后端
    - Spring
    - maven
---


# Spring IOC原理与样例(使用maven)

## 1. 概论Sammary


### 1.1 IOC的背景和原理

> 在采用面向对象方法设计的软件系统中，底层实现都是由N个对象组成的，所有的对象通过彼此的合作，最终实现系统的业务逻辑。即软件系统中对象之间的耦合，对象A和对象B之间有关联，对象B又和对象C有依赖关系，这样对象和对象之间有着复杂的依赖关系，所以才有了控制反转这个理论。

### 1.2 什么是IOC（控制反转）

> (1).IoC是Inversion of Control的缩写，有的翻译成“控制反转”，还有翻译成为“控制反向”或者“控制倒置”。

> (2).1996年，Michael Mattson在一篇有关探讨面向对象框架的文章中，首先提出了IoC 这个概念。简单来说就是把复杂系统分解成相互合作的对象，这些对象类通过封装以后，内部实现对外部是透明的，从而降低了解决问题的复杂度，而且可以灵活地被重用和扩展。IoC理论提出的观点大体是这样的：借助于“第三方”实现具有依赖关系的对象之间的解耦

#### 举个例子

在使用IOC解耦以前，对象之间是紧密依赖的，和齿轮一样，一旦其中一个对象需要改动就可能一发而动全身：

![noIOC](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/img1.png)

但是使用IOC解耦以后呢，齿轮之间通过一个第三方“中介”（IOC容器）进行协调，实现了解耦：

![useIOC](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/img2.png)

#### 再举个例子
电脑主机和硬盘：`主机和硬盘`（耦合的两个对象）是耦合在一起的，要是想实现他们之间的解耦怎么做呢？

那就是——使用`移动硬盘`和`支持移动硬盘的主机`（解耦后的对象），而`人（也就是IOC容器)`决定要什么数据时，就给主机插上哪一个移动硬盘，这样我们就不需要在想获取其他地方的数据时专门把主机拆了换硬盘，从而实现主机和硬盘的解耦。


## 2. Spring IOC 简单样例

### 2.1 样例讲解

关于样例的讲解可以[移步此处](http://www.open-open.com/lib/view/open1326850984030.html)

> 以下针对此样例讲解具体操作

### 2.2 样例实现

首先新建一个maven项目

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102511.png)

这个使用默认
![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102247.png)

注意这里新建的是普通的maven项目，**不是**一个`webApp`

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102307.png)

输入项目名称

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102336.png)

新建了maven项目以后，配置`pom.xml`文件，加入spring依赖

```xml
<!-- spring 依赖 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>3.2.8.RELEASE</version>
</dependency>

```

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102611.png)

使用样例的代码建立相关的接口和类

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102829.png)

建立完接口、类以后的文件结构是这样的

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928102921.png)

在`src---main`文件夹下新建一个`source fold`, 命名为`resources`

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928104519.png)

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928104604.png)

在resources文件夹下新建一个`file` 命名为`beans.xml`

注意，开头的给的样例里面的`beans.xml`有语法错误，而如下的`beans.xml`经过测试证明有效

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="springQuizMaster" class="com.vaannila.SpringQuizMaster"></bean>
	<bean id="strutsQuizMaster" class="com.vaannila.StrutsQuizMaster"></bean>
	<bean id="quizMasterService" class="com.vaannila.QuizMasterService">

		<property name="quizMaster" ref="springQuizMaster">
		</property>
	</bean>
</beans>
```

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928104712.png)

然后在入口程序所在的类`QuikProgram.java`里面引用这个配置文件：`beans.xml`

```java
package com.vaannila;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class QuizProgram {
	public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");//读取beans.xml中的内容
		QuizMasterService quizMasterService = (QuizMasterService)ctx.getBean("quizMasterService");

		quizMasterService.askQuestion();
	}
}
```

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928104756.png)

运行程序

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928104827.png)

要使用`StructQuikMastrer`就只需要在`beans.xml`里面修改配置为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="springQuizMaster" class="com.vaannila.SpringQuizMaster"></bean>
	<bean id="strutsQuizMaster" class="com.vaannila.StrutsQuizMaster"></bean>
	<bean id="quizMasterService" class="com.vaannila.QuizMasterService">
		<!-- 修改这里的ref属性 -->
		<!-- <property name="quizMaster" ref="springQuizMaster"> -->
		<property name="quizMaster" ref="strutsQuizMaster">
		</property>
	</bean>
</beans>
```
运行结果

![image](https://raw.githubusercontent.com/youngdou/youngdou.github.io/master/img/post_imag/2016-09-02/20160928104906.png)




## 3. 总结


### 3.1 关于代码

- 关于以上讲解的代码托管在我的github:[IOC_test](https://github.com/youngdou/code_for_course_DistributedComputing/tree/master/IOC_test)

- 参考文章
[Spring框架中IoC(控制反转)的原理](http://blog.csdn.net/u012561176/article/details/45974315)

### 3.2 想法

关于java web开发，有许多好玩有趣，发人深省的架构。有人说，艺术来源于生活。前段时间看过日本设计师原研哉的书，发现。其实吧，设计，无论是关乎哪个方面的设计，都来自于生活并且服务与生活。对生活越热爱，对万物也就越敬畏，就越懂得万物之灵动。