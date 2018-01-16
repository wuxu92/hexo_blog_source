---
date: 2015/09/02 12:34:50
title: php使用openssl加密数据
categories:
- programming
tags:
- php
- openssl
- security
---

最近在为公司的游戏接入多家支付平台，其中多次使用到openssl模块验证数据的签名，之前在做阿里支付的时候也有做支付的回调，但是当时直接套一个sdk代码，没有仔细研究，这里记录一下在php中使用openssl加密数据和验证数据签名的方法。

这里主要包括两个部分，一是直接加密数据，把原来的数据使用密钥加密后传输，在接收端使用密钥解密出数据，这种方法被少数的支付平台使用，比如安智？另一种更常用的是把传输的数据做一个数字签名，数据本身使用明文传输，接收方按照约定的方式使用接受的数据计算一个签名，然后比照接受的签名和计算的签名是否相同。
这两种方法各有优势，按需使用。下面分别介绍。

## 生成密钥对 ##
首先确保你的php环境开启了openssl模块，这个具体不细说。建议在编译php时静态编译好。
生成密钥对有两种方法，一是使用php提供的方法generate一对密钥，另一种使用openssl命令行生成。
### 使用php函数生成密钥对 ###
openssl模块提供了很多openssl相关的函数，[参考手册](http://php.net/manual/en/ref.openssl.php "http://php.net/manual/en/ref.openssl.php") 生成密钥对的方法如下：

```php
$privateKey = openssl_pkey_new([
  'private_key_bits' => 2048,  // private key的大小
  'private_key_type' => OPENSSL_KEYTYPE_RSA,
]);

openssl_pkey_export_to_file($privateKey, 'php-private.key');
$key = openssl_pkey_get_details($privateKey);
file_put_contents('php-public.key', $key['key']);

openssl_free_key($privateKey); // 释放资源
```
<!-- more -->
要注意，openssl key是一种资源类型，在使用完后记得释放资源。

### 使用openssl命令生成 ###
linux的openssl包可以直接用来生成密钥对。

```bash
opensll genrsa 512 > private.key
openssl rsa -pubout < private.key > public.key
```
这样就生成了一对密钥对。

## 使用密钥对加密数据 ##

使用第一步的php函数生成的公钥对一段明文进行分段(chunk)再分段加密，（实际使用中也可以直接全部文本加密）：

```
$plain = 'this data will be encrypted for transform dolot virendrachadr dark';
echo 'plian text: ' . $plain;
$plain = gzcompress($plain); // compress data
$pubkeyStr = file_get_contents('./php-public.key');
$publicKey = openssl_pkey_get_public($pubkeyStr);

$p_key = openssl_pkey_get_details($publicKey);
$chunkSize = ceil($p_key['bits'] / 8) -11; // 这里不知道为什么要-11，后面追加解释

$output = '';

while ($plain) {
  $chunk = substr($plain, 0, $chunkSize);
  $plain = substr($plain, $chunkSize);

  $encrypted = '';
  if ( !openssl_public_encrypt($chunk, $encrypted, $publicKey)) {
    die("failed to encrypt data");
  }
  $output .= $encrypted;
}
openssl_free_key($publicKey);
$output = base64_encode($output);
echo 'encrypted: ' . ($output);
file_put_contents('./enc.data', $output);

```
运行后的输出类似于(生成的公钥不一样，加密的结果也会不一样)：

```
plian text: this data will be encrypted for transform dolot virendrachadr dark
encrypted: fal8QtGky0+PbEQ43s8enksH3Wkf39NyeemyjwdtAZvgCnCjF7ZDh6cGSy7ROaN9ite/lfTzJwupiZtXqZojWWzIqq+J3P/58LZogRgWACbRBevq5JF1XmBiQCNDdRCWaBAC3HToUfryh9+0OzxN5I4Txk+/+j4WdQpNyuUMJbNGlNXUdGfL7I6Hw07DAooqDLKGYCu9bfM8aOVn6MASmdegQzYw58YtbfPZIfSAKB35JJLlVK5mJX/4g/GIzFdbj3s9pQL7Xhs0+y2oi5GNsAO45vxrHE9xu+SM8c0/jAQKjXjm5KACsCUUw2zAi/G/g6cUsJAQfrbHKdpwcBp5JQ==
```

**追加**： 关于什么加密时分片要减去11：2048位密钥加密的数据输出应该是2048bit，也就是256byte。在[函数的官方文档](http://php.net/manual/zh/function.openssl-public-encrypt.php "http://php.net/manual/zh/function.openssl-public-encrypt.php")中[第一个User Notes](http://php.net/manual/zh/function.openssl-public-encrypt.php#55901 "http://php.net/manual/zh/function.openssl-public-encrypt.php#55901")里提到了：

> However, the PKCS#1 standard, which OpenSSL uses, specifies a padding scheme (so you can encrypt smaller quantities without losing security), and that padding scheme takes a minimum of 11 bytes (it will be longer if the value you're encrypting is smaller)

这个note里面还提到了能加密字符串长度的限制，2048位密钥加密的data长度限制为： `2048/8 - 11`了。使用其他大小的密钥时也减去11就可以了。
## 解密数据 ##
使用私钥对数据进行解密：

```php
$keyStr = file_get_contents('./php-private.key');
if (!$privateKey = openssl_pkey_get_private($keyStr)) {
  die('get private key failed');
}

$encrypted = file_get_contents('./enc.data');
echo 'encrypted data: ' . $encrypted;

$encrypted = base64_decode($encrypted);

$p_key = openssl_pkey_get_details($privateKey);
$chunkSize = ceil($p_key['bits'] / 8);
$output = '';

while ($encrypted) {
  $chunk = substr($encrypted, 0, $chunkSize);
  $encrypted = substr($encrypted, $chunkSize);
  $decryptd = '';
  if (!openssl_private_decrypt($chunk, $decryptd, $privateKey)) {
    die('failed to decrypt data');
  }
  $output .= $decryptd;
}
openssl_free_key($privateKey);
$output = gzuncompress($output);
echo "\ndecrypted data: " . $output;
```
解密之后的数据就是我们之前加密的数据啦：

```
encrypted data: fal8QtGky0+PbEQ43s8enksH3Wkf39NyeemyjwdtAZvgCnCjF7ZDh6cGSy7ROaN9ite/lfTzJwupiZtXqZojWWzIqq+J3P/58LZogRgWACbRBevq5JF1XmBiQCNDdRCWaBAC3HToUfryh9+0OzxN5I4Txk+/+j4WdQpNyuUMJbNGlNXUdGfL7I6Hw07DAooqDLKGYCu9bfM8aOVn6MASmdegQzYw58YtbfPZIfSAKB35JJLlVK5mJX/4g/GIzFdbj3s9pQL7Xhs0+y2oi5GNsAO45vxrHE9xu+SM8c0/jAQKjXjm5KACsCUUw2zAi/G/g6cUsJAQfrbHKdpwcBp5JQ==
decrypted data: this data will be encrypted for transform dolot virendrachadr dark
```

## 数字签名 ##
使用完全加密的数据进行传输的好处是更加安全，但是计算更加复杂，需要传输的数据也更多，更常用的方式只是对要传输的数据做一个数字签名，在接收端对接收到的数据进行一个签名运算，只要客户端计算的签名和接受的的签名一样就可以认为收到的数据没有被篡改过。

计算签名使用openssl提供的`openssl_sign()`，签名验证使用`openssl_verify()`
这两个函数的函数签名为：

```
bool openssl_sign ( string $data , string &$signature , mixed $priv_key_id [, mixed $signature_alg = OPENSSL_ALGO_SHA1 ] )
int openssl_verify ( string $data , string $signature , mixed $pub_key_id [, mixed $signature_alg = OPENSSL_ALGO_SHA1 ] )
```
通过参数比较容易理解函数的使用，sign函数第一个函数是一个字符串，所以对数组，对象等签名需要使用`json_encode`或者`base64_encode`等函数编码一下；第二个参数是`&$signature`就是函数会把对数据`$data`的签名保存在`$signature`变量。

注意返回值，第一个函数是bool值，第二个是int，1表示签名验证通过， 0表示签名不正确，-1表示发生错误。
一个示例：

```php
$publicKey = file_get_contents('./php-public.key');
$privateKey = file_get_contents('./php-private.key');

$data = [
  'orderId' => 100002,
  'pay_time' => '2015-09-02 10:10:10'
];
$signature = '';
openssl_sign(json_encode($data), $signature, $privateKey);
echo 'sign is: ' . base64_encode($signature);

$verify = openssl_verify(json_encode($data), $signature, $publicKey);

echo "\nverify result: $verify";
```
上面使用的签名密钥是2048位的所以签名会比较长,实际使用的时候使用512位甚至128位的比较多：

```
sign is: cT3/wbc2l7nKkb0BR6ZhvrDychPDVrl+1Dsjo5IDzWvuSsd3H17F5S1F6C2BwXLe6UMqigCsEplkuBvp0J1ZW3utrNAZLzWvnaMHXu0oiBrqp0Mgud2qcjcGvpF10Fs70OqyPlf2d0v0YOSg/vZ7MAeZNPOEYgcxVhsol9WCyboFyuqUNVPyZb629M/fDMofemwVBGxc/u/+NIRxFFDawPaIwPauuengrEs4sTcL12Yyx+l4pW6VQ1yXOAeBgg43SEcNr7LoKV7ALGhcAws2gIhHkUgfqfKobq19e02j4Zi+ZouorlgVDu8Fst0nejFvze1vQfDgEpCNODpzNE51yg==
verify result: 1
```
上面的例子把签名和验签放在一起了，实际使用中，一般签名在数据发送方进行，验签在接收端进行。

### 注意 ###
对数据进行签名时要注意一点，一般发送方通过post把数据发送给接收方，在接收方收到的post数据的顺序并不能保证和发送发签名时一样，所以要约定到post数组的键的顺序，一般在签名前进行```ksort($data)```。当然如果使用raw_post数据（```php://input```）那就没关系了。

**参考：**

- [https://www.virendrachandak.com/techtalk/encryption-using-php-openssl/](https://www.virendrachandak.com/techtalk/encryption-using-php-openssl/ "https://www.virendrachandak.com/techtalk/encryption-using-php-openssl/")
- [http://blog.csdn.net/samxx8/article/details/47003561](http://blog.csdn.net/samxx8/article/details/47003561 "http://blog.csdn.net/samxx8/article/details/47003561")
