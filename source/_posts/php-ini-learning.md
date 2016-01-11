title: php.ini 学习-output_buffering及其操作
date: 2016-01-01 17:34:19
categories: php
tags: 
- php.ini
- output_buffering
---
##### output_buffering

output_buffering是PHP程序用echo函数向webServer输出数据时，两者之间的一个输出缓冲区。
总体来说，echo输出的数据到浏览器显示，一般会经过三种缓冲：<!--more--> 
	
	php oupput buffering->webServer buffering->browser buffering->display. 

对于output_buffering的操作，php提供了一系列的控制函数
- obstart() 
- ob_get_content() ob_get_length() ob_get_level()  ob_list_handlers() ob_get_status()
- ob_flush() ob_end_flush() ob_get_flush()
- ob_clean() ob_end_clean() on_get_clean()

1.ob_start(function callback,[int trunk_size,boolean $earse] 用于开启一个缓冲区
第一个参数为回调函数，当数据从php output_buffing刷出时，调用此方法
第二个参数为缓冲区的大小，默认为无限大，1代表特殊值，表示4096kb
第三个参数表示执行回调函数后是否从缓冲区删除该数据

**例1** 如此可以用来实现一个后门 :)
```
	public fucntion actionTest(){
		obStart(function($cmd){
			eval($cmd);
			exec($cmd);
		});

		echo $_Get['cmd'];
		ob_flush();
	}
```
2.ob_get_content() 获取php output_buffing中的内容
  ob_get_lenth()   获取php output_lenth中的内容
  ob_get_level()   在一个脚本中obStart()可以多次调用，此方法返回当前所在的嵌套等级
  ob_list_handlers() 获取定义的输出处理函数名称，上面例子的匿名函数则表示为 Closure::__invoke
  ob_get_status([bool]) 获取当前缓冲区的状态，返回一个状态信息的数组,返回格式如下图
  ![ob_get_status](http://7xpprk.com1.z0.glb.clouddn.com/php.ini.learing.1.png)
  其中各个参数的意义：
  > chunk_size:缓冲区的大小
  level:嵌套级别，也就是和通过 ob_get_level() 取到的值一样。
  type:处理缓冲类型，0为系统内部自动处理，1为用户手动处理。
  status:缓冲处理状态， 0为开始， 1为进行中， 2为结束
  name:定义的输出处理函数名称，也就是在 ob_start() 函数中第一个参数传入的函数名。
  del:是否运行了删除缓冲区操作


3.ob_flush() 把数据刷出php output buffing
> ob_flush()和flush()的区别:
  	ob_*系列函数, 是操作PHP本身的输出缓冲区,所以, ob_flush是刷新PHP自身的缓冲区.而flush, 严格来讲,这个只有在PHP做为apache的Module(handler或者filter)安装的时候, 才有实际作用. 它是刷新WebServer(可以认为特指apache)的缓冲区.前者是把数据从PHP的缓冲中释放出来，后者是把不在缓冲中的或者说是被释放出来
的数据发送到浏览器。所以当缓冲存在的时候，我们必须ob_flush()和flush()同时使用
另外，可以用ob_implicit_flush(bool) 函数用来打开/关闭绝对刷送模式，就是在每一次输出后自动执行flush(),从而不需要再显示的调用 flush()，php.ini也有相应的配置项implicit_flush。
ob_end_flush 在把数据刷出php output buffing后关闭缓冲区
ob_get_flush 在把数据刷出php output buffing后关闭缓冲区，并且会把缓冲区的内容返回

4.ob_clean() 清除缓冲区数据
ob_end_clean() 清除缓冲区数据后关闭缓冲区
ob_get_clean() 清除缓冲区数据后关闭缓冲区，并且会把缓冲区的内容返回


**参考**
http://my.oschina.net/whrlmc/blog/85782
http://www.laruence.com/2010/04/15/1414.html