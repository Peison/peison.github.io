title: 使用/dev/random和/dev/urandom生成随机数
date: 2016-05-06 15:35:50
categories: php
tags:
- RGN
- 随机数
---
最近在研读Yii2.0框架的源代码，发现在Security组件中，有一个获取随机数的方法，使用了操作系统
提供的产生随机数的设备：/dev/random和/dev/urandom
<!--more-->
网上搜索，知道/dev/random设备，存储着系统当前运行的环境的实时数据。它可以看作是系统某个时候的唯一值数据，因此可以用作随机数元数据。我们可以通过文件读取方式，读得里面数据。/dev/urandom这个设备数据与random里面一样。只是，它是非阻塞的随机数发生器，读取操作不会产生阻塞。

作者在注释中描述了实现方法的原理：
>  		/*
         * Strategy
         *
         * The most common platform is Linux, on which /dev/urandom is the best choice. Many other OSs
         * implement a device called /dev/urandom for Linux compat and it is good too. So if there is
         * a /dev/urandom then it is our first choice regardless of OS.
         *
         * Nearly all other modern Unix-like systems (the BSDs, Unixes and OS X) have a /dev/random
         * that is a good choice. If we didn't get bytes from /dev/urandom then we try this next but
         * only if the system is not Linux. Do not try to read /dev/random on Linux.
         *
         * Finally, OpenSSL can supply CSPR bytes. It is our last resort. On Windows this reads from
         * CryptGenRandom, which is the right thing to do. On other systems that don't have a Unix-like
         * /dev/urandom, it will deliver bytes from its own CSPRNG that is seeded from kernel sources
         * of randomness. Even though it is fast, we don't generally prefer OpenSSL over /dev/urandom
         * because an RNG in user space memory is undesirable.
         *
         * For background, see http://sockpuppet.org/blog/2014/02/25/safely-generate-random-numbers/
         */


下面是具体的代码：
```
	public function generateRandomKey($length = 32)
    {   
        $bytes = '';

        // 如果当前的操作系统中存在“/dev/urandom”文件，则从改文件中读取指定长度的随机字符串
        if (@file_exists('/dev/urandom')) {
            $handle = fopen('/dev/urandom', 'r');
            if ($handle !== false) {
                $bytes .= fread($handle, $length);
                fclose($handle);
            }
        }

        if (StringHelper::byteLength($bytes) >= $length) {
            return StringHelper::byteSubstr($bytes, 0, $length);
        }

        // 如果没有“/dev/urandom”文件，但是存在“/dev/random”文件并且当前系统不是Linux系统，则从“/dev/random”文件读取
        if (PHP_OS !== 'Linux' && @file_exists('/dev/random')) {
            $handle = fopen('/dev/random', 'r');
            if ($handle !== false) {
                $bytes .= fread($handle, $length);
                fclose($handle);
            }
        }

        if (StringHelper::byteLength($bytes) >= $length) {
            return StringHelper::byteSubstr($bytes, 0, $length);
        }

        if (!extension_loaded('openssl')) {
            throw new InvalidConfigException('The OpenSSL PHP extension is not installed.');
        }

        //如果两个文件都没有，则调用openssl生成一个伪随机窜
        $bytes .= openssl_random_pseudo_bytes($length, $cryptoStrong);

        if (StringHelper::byteLength($bytes) < $length || !$cryptoStrong) {
            throw new Exception('Unable to generate random bytes.');
        }

        return StringHelper::byteSubstr($bytes, 0, $length);
    }
```