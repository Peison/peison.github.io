title: 转：php mcrypt 拓展使用
date: 2016-03-12 16:02:15
categories: php
tags: 
- 加密
- mcrypt
---

> Author:fleshwound
From:[Security Matrix](http://bbs.php100.com/read-htm-tid-80157.html)
<!--more-->

1. 什么是对称密码算法
   一个密码系统需要有五个要素部分组成（密文，明文，加密算法，解密算法，密钥）。区别对称与非对称密码算法的关键在于解密密钥是否会等于加密密钥，如果相等，那么就是对称的，不相等就是非对称的。像DES，AES都属于对称密码算法，而RSA,ECC都属于非对称的公钥密码算法。一般而言，对称密码算法加解密速度较快，通常是用来进行数据加密，而非对称的公钥加密算法加解密速度较慢，通常是用来进行数字签名和构造复杂的密码协议。坊间传说的某某公钥加密算法比某某对称加密算法更好更安全实属荒谬，这就好比讨论你的手和脚那个更有用。

2. 什么叫做对称密码算法的加密模式
这个是很多开发人员搞得不太清楚的东西，毕竟大部分PHP开发者未必都接触过密码学方面的专业培训，因此看到PHP的相关函数感觉望而生畏，不知所措。所谓加密模式就是加密任意长度的消息的使用密码算法的方法。通常对称密码算法都会提供至少四种基本工作模式，其中两组为块模式（ECB和CBC），两组是流模式(CFB和OFB)。打个通俗的比方，密码算法函数就好像个机器，明文就好像原材料，机器不可能一次处理所有明文，那么块模式的策略就是把明文进行分组，然后按照一定的规则放入机器做处理，然后得到一组密文。ECB模式就是将消息分成相互独立的块，每一块都用同一算法和密钥加密（比如使用DES）,块与块之间没有关联。而CBC也是将消息分成相互独立块，但是加密的时候，做了点手脚，使得下一块的密文和上一块的密文有关系，分块加密前与上一块的密文进行异或运算，然后在用同一算法和密钥加密，由于第一块明文没有上一块的密文，所以必须人工指定一个，称之为初始化向量。而CFB和OFB模式把明文看作是二进制流，实际上也是分块的，它们都需要一个初始化向量，不断地生成伪随机二进制流，然后与明文分组进行异或运算，而密文是否参与运算是CFB和OFB之间的区别所在。

3. PHP中的对称密码算法函数
PHP的各个版本都提供了相应的对称密码算法函数，所以PHP的开发者们完全没必要自己山寨一个，请相信这些提供的对称密码算法函数绝对比你自己设计的更强大。目前为止，不绝于耳的是各种颜色的客（还有祖传的）据说能够几分钟或者秒破一个密码算法，其实这是误解，除了在看雪论坛上看到几个直接针对密钥的暴力破解实例外，偶在网络上很少见过数学上能攻破一个密码算法的人。之所以系统或软件被破解，是因为密码算法使用不当或者攻击者绕过了密码算法机制。PHP的主要对称密码算法函数来自于Mcrypt函数族。当然这个函数族是PHP的可选部件，需要下载libmcrypt-x.x.tar.gz并在PHP.INI中进行配置。你可以通过phpinfo()来看看自己配置的环境是否支持这些函数。

```
	mcrypt support: enabled
	Version:  2.5.8  
	Api No: 20021217  
	Supported ciphers:  cast-128 gost rijndael-128 twofish arcfour
			cast-256 loki97 rijndael-192 saferplus wake 
			blowfish-compat des rijndael-256 serpent xtea 
			blowfish enigma rc2 tripledes  
	Supported modes:  cbc cfb ctr ecb ncfb nofb ofb stream  
```

首先我们看到这个函数mcrypt_module_open，原型是：
```
	resource mcrypt_module_open ( string algorithm, string algorithm_directory,
	 			string mode, string mode_directory)
```

这个函数的作用实际上是创建一个使用对称加密算法的工作环境，告诉PHP要使用何种算法（第一个参数），算法路径在哪里（第二个参数），采用何种工作模式（第三个参数），工作模式描述的路径在哪里(第四个参数).实际使用中我们一般不需要设置第二个参数和第四个参数。现在我们看个例子：

```
	<?php
	//php拓展中的加密算法研究

	//列举算法种类
	$algorithms = mcrypt_list_algorithms();
	sort($algorithms);
	print_r($algorithms);

	//列举加密模式种类
	$models = mcrypt_list_modes();
	sort($models);
	print_r($models);

	//生成密钥
	$key = md5('<3pfg39a>asdkw4=weqwazS98asd*34&34%');
	echo $key.PHP_EOL;

	//开启加密模块
	$td = mcrypt_module_open(MCRYPT_DES,'',MCRYPT_MODE_ECB,'');

	//生成初始向量
	$iv = mcrypt_create_iv(mcrypt_enc_get_iv_size($td),MCRYPT_RAND);

	//获取加密算法密钥的最大长度
	$keySize = mcrypt_enc_get_key_size($td);
	$key = substr($key,0,$keySize);

	echo $keySize.PHP_EOL;

	//初始化加密算法
	mcrypt_generic_init($td,$key,$iv);//初始化加密算法，当算法模式是ecb时候，会自动忽略iv
	//加密
	$encrypt =  mcrypt_generic($td,'i love you,lindsey');
	//执行清理工作，释放资源
	mcrypt_generic_deinit($td);

	echo $encrypt.PHP_EOL;

	//解密，由于$iv没有被清掉，所以我们还可以继续用这个。实际使用中一定要保存起来
	$iv = mcrypt_create_iv(mcrypt_enc_get_iv_size($td),MCRYPT_RAND);
	//初始化加密算法
	mcrypt_generic_init($td,$key,$iv);
	//解密
	$decrypt = mdecrypt_generic($td,$encrypt);
	//执行清理工作,释放资源
	mcrypt_generic_deinit($td);

	echo $decrypt.PHP_EOL;

	//关闭加密算法
	mcrypt_module_close($td);
```

- 这里有几个函数，第一个是mcrypt_createiv，这个是用来创建一个随机的初始化向量IV，四种基本模式中，
除了ECB，均要有初始化向量IV。
- mcrypt_generic_init函数用来初始化密码算法对象，包括具体算法的句柄，密钥和初始化向量。
当算法模式是ECB时，该函数会自动忽略初始化向量IV。
- 接下来利用mcrypt_generic执行加密。加密完成后利用mcrypt_generic_deinit和mcrypt_module_close来释放环境。
- 值得注意的是，由于IV生成的时候使用了随机数,
所以每次是不一样的，也会导致密文不一样。因此除了ECB模式外，其它三种模式均要把IV记下了，才能进行解密。
解密过程和加密过程是一样的，只是函数mcrypt_generic换成了mdecrypt_generic，请注意，别眼花，注意他们的书写上的区别，前者是加密，后者是解密。这个过程是PHP中标准的使用对称密码算法的过程。
- PHP中有些非常陌生的密码算法，比如gost，除非专业搞密码算法的，一般很少人会知道这个算法的具体内容。

我们可以使用以下PHP的算法信息函数来了解一个算法：
```
	mcrypt_enc_get_algorithms_name –获取算法具体名称
	mcrypt_enc_get_block_size – 获取算法的分组大小
	mcrypt_enc_get_iv_size – 获取算法的IV初始化向量大小
	mcrypt_enc_get_key_size – 获取算法支持的最大密钥长度
	mcrypt_enc_get_modes_name – 获取算法支持的加密模式
	mcrypt_enc_get_supported_key_sizes –返回一个数组，里面有算法支持的各种密钥长度
```

使用方法如下：
```
<?php
    $td = mcrypt_module_open (MCRYPT_BLOWFISH, '', 'ecb', '');
    echo mcrypt_enc_get_algorithms_name($td). "\n";
    echo mcrypt_enc_get_block_size($td). "\n";
    echo mcrypt_enc_get_iv_size($td). "\n";
    echo mcrypt_enc_get_key_size($td). "\n";
    echo mcrypt_enc_get_modes_name($td). "\n";
    //结果：Blowfish 8 8 56 ECB
?>
```

PHP中除了上述用法之外，还有一种加密解密的用法。依赖于函数mcrypt_encrypt和mcrypt_decrypt,前者用于加密，后者用于解密。其函数原型为：

```
	string mcrypt_encrypt ( string cipher, string key, string data, string mode [, string iv])
	string mcrypt_decrypt ( string cipher, string key, string data, string mode [, string iv])
```

我们很容易发现，两个函数的参数都是完全一样的，实际上也只有一样才能正常的加密和解密。这种用法与前述函数用法的区别在于这两个函数把所有的算法信息当作为自己的参数了，习惯于传统PHP开发的人会觉得这个更加易用。先看个代码：

```
    <?php
     //获取该算法在CBC模式下的IV向量大小
    $iv_size = mcrypt_get_iv_size(MCRYPT_RIJNDAEL_256, MCRYPT_MODE_CBC); 
    $iv = mcrypt_create_iv($iv_size, MCRYPT_RAND); //创建一个初始化IV向量
    $key = "security matrix";   //密钥
    $plaintext = "you should arrive here in eight clock"; //明文
    //直接执行加密
    $ciphertext = mcrypt_encrypt(MCRYPT_RIJNDAEL_256, $key, $plaintext, MCRYPT_MODE_CBC, $iv); 
    echo  "明文是：$plaintext.<br> 密文是: $ciphertext";
    //执行解密
    $outtext = mcrypt_decrypt(MCRYPT_RIJNDAEL_256, $key,$ciphertext, MCRYPT_MODE_CBC, $iv); 
    echo "<br>";
    echo  "解密后的明文是:  $outtext<br>";

//结果：
//明文是：you should arrive here in eight clock.
//密文是: p 蜉?~嵁'mh!Bl廀'Es??{ ?e?斤舉* YC¬濤溋C海Rc敻J 偰k躕"蟸v勸
//解密后的明文是: you should arrive here in eight clock
?>
```
