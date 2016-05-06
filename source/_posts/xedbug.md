title: phpStrom xedbug环境配置
date: 2016-02-16 11:36:28
categories: php
tags:
- xdebug
---
通常在调试PHP时，都是使用var_dump(),print_r()等输出中间变量，查找问题。
而使用xdebug，则可以对PHP程序进行单步调试，在代码逻辑比较复杂的情况下，还是挺方便的。
另外，一些代码使用了比较多的反射机制，阅读起来比较困难，使用debug一步步跟进阅读代码也是个不错的选择。
<!--more-->
这里只介绍在phpStrom配置xdebug，其他IDE我也没有折腾过，不是很清楚，具体步骤如下：
#### 1.下载xdebug拓展
在 https://xdebug.org/download.php 中找到对应版本的xdebug拓展，并保存到本地的PHP目录下
在本地的php.ini文件末尾，加上如下配置
```
	; XDEBUG Extension
	zend_extension = "D:/wamp/bin/php/php5.5.12/zend_ext/php_xdebug-2.2.5-5.5-vc11-x86_64.dll"
	
	[xdebug]
	xdebug.remote_enable = off
	xdebug.profiler_enable = off
	xdebug.profiler_enable_trigger = off
	xdebug.profiler_output_name = cachegrind.out.%t.%p
	xdebug.profiler_output_dir = "D:/wamp/tmp"
	xdebug.show_local_vars=0

	xdebug.remote_enable = On
	xdebug.remote_handler = dbgp
	xdebug.remote_host= localhost 
	xdebug.remote_port = 9000 #端口号，可以自定义
	xdebug.idekey = PHPSTORM  #key值，可以自定义
```

#### 2.配置PHPStrom
打开File>>Setting,找到languages & frameworks,设置xdebug port为9000
![](http://7xpprk.com1.z0.glb.clouddn.com/1.jpg)
再进入xdebug下DBGp proxy,填写相应配置（对应上面的php.ini的配置）
![](http://7xpprk.com1.z0.glb.clouddn.com/2.jpg)
再按下图，配置本地的 web server~

- 点击Edit Configurations
![](http://7xpprk.com1.z0.glb.clouddn.com/3.jpg)


- 添加PHP WEB Application
![](http://7xpprk.com1.z0.glb.clouddn.com/4.jpg)


- 添加web server
![](http://7xpprk.com1.z0.glb.clouddn.com/5.jpg)


- 假设本地虚拟主机域名为 www.yii2.com 服务器的端口为80
![](http://7xpprk.com1.z0.glb.clouddn.com/6.jpg)


#### 3.测试一下
打断点，点击debug按钮
![](http://7xpprk.com1.z0.glb.clouddn.com/7.jpg)
可以进入各种中间变量的值，代表就以及成功了
![](http://7xpprk.com1.z0.glb.clouddn.com/8.jpg)

#### 4.配合浏览器使用
使用chrome浏览器，可以安装一个叫Xdebug helper插件，可以更加方便地进行调试。
浏览器安装好xdebug后，填写IDE KEY
![](http://7xpprk.com1.z0.glb.clouddn.com/9.jpg)

并且打开PHPStrom中的这个按钮
![](http://7xpprk.com1.z0.glb.clouddn.com/10.jpg)

在浏览器中地址栏开启xdebug helper，回车，就可以进入打了断点的代码中
![](http://7xpprk.com1.z0.glb.clouddn.com/11.jpg)
是不是很方便 ^_^