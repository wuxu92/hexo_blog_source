---
date: 2015/08/30 12:34:50
title: Go by Example(六)：杂烩2
categories:
- programming
tags:
- golang
- go
---


## 随机数/Rand ##
在之前的练习中，已经使用过随机数相关的函数了，比如随机休眠N毫秒：

```
time.Sleep(time.Milliseond * time.Duration(rand.Intn(500) ))
```
math/rand包提供了很多方法，上面的`Intn(n int)`方法就是放回n以内整数这是最常用的函数之一。另外常用的方法包括：

```
rand.Float64()  // 返回0-1之间的一个浮点数
rand.Int31()
rand.Uint32()
```
和其他平台的随机数生成器一样，rand包提供的随机数序列也是伪随机数，如果不传入一个不同的种子/seed，得到的随机数序列将是确定的。我们常把时间作为随机数种子传入方法：

```
rs := rand.NewSource(time.Now().UnixNano())
r := rand.New(rs)
// r此时是一个新的随机数生成器
// 正式使用应该这样
r.Intn(100)
```

## strconv/数字与字符串转换 ##
strconv包提供了字符串与数字类型的转换，这是一些很常用的功能，比如从"123"到123的转换，"1.23"转换到1.23;特别是在与客户端通信，使用json，或者处理从数据库查询的数据的时候，经常需要这类转换。

```golang
// parseFloat原型
func ParseFloat(s string, bitSize int) (f float64, err error)
f,_ := strconv.ParseFloat("1.23", 64)   // 转换到64位Float，不处理转换的错误

// parseInt的原型， base从2-36
func ParseInt(s string, base int, bitSize int) (i int64, err error)

// 更常用的
k,_ := strconv.Atoi("123")  // k=123
_,e := strconv.Atoi("wrongFormat")  // will return error
```
其中最常用的是`Atoi(s tring)`和`Itoa(i int)`;strconv包提供的方法大部分会有两个返回值，第一个是转换的结果，第二个是是否转换成功的error。

## URL相关 ##
net/url包提供了对URL处理的方法，可以把一个字符转解析成一个url变量，通过提供的属性获取URL的各个部分。
我们自导一个完整的URL的模式是这样的： 

```
scheme://[username:password@]host/path?querystring#fragement_id
```
其中的host包括domain和port。

url包提供了把一个字符串解析成url，并提供访问上面模式中各个元素的方法:

```
import "net/url"

urlStr := "https://gobyexample.com/url-parsing?from=goole#url-parsing"
u, err := url.Parse(urlStr)
if err != nil {  fmt.Println(err)
}
fmt.Println(u.Scheme,u.Host, u.Path, u.Fragment, u.RawQuery)
host, port, _ := net.SplitHostPort(u.Host)

user := u.User
username := "null"
if user != nil {
  username = user.Username()  // user.Password()
}
fmt.Println("username", username)
// 解析queryString
query, _ := url.ParseQuery(u.RawQuery)
fmt.Println(query, query["from"][0])
```
> The parsed query param maps are from strings to slices of strings, so index into [0] if you only want the first value.

注意querystring解析出来的map是一个建对应一个slice的。不是一一对应的字符串，这是因为请求时可能有相同的key，一般只取第一个。

## Hash/SHA1 ##
Go的crypto包及其子包中包括多种hash函数，我们最常用的MD5和SHA1都包含在里面。各种hash算法实现在各个子包中，比如md5在 `crypto/md5`, sha1在 `crypto/sha1`。
子包列表和简介：


|Name |	Synopsis |
|-----|----------|
|aes |	Package aes implements AES encryption (formerly Rijndael), as defined in U.S. Federal Information Processing Standards Publication 197. |
|cipher |	Package cipher implements standard block cipher modes that can be wrapped around low-level block cipher implementations. |
|des |	Package des implements the Data Encryption Standard (DES) and the Triple Data Encryption Algorithm (TDEA) as defined in U.S. Federal Information Processing Standards Publication 46-3. |
|dsa |	Package dsa implements the Digital Signature Algorithm, as defined in FIPS 186-3. |
|ecdsa |	Package ecdsa implements the Elliptic Curve Digital Signature Algorithm, as defined in FIPS 186-3. |
|elliptic |	Package elliptic implements several standard elliptic curves over prime fields. |
|hmac |	Package hmac implements the Keyed-Hash Message Authentication Code (HMAC) as defined in U.S. Federal Information Processing Standards Publication 198. |
|md5 |	Package md5 implements the MD5 hash algorithm as defined in RFC 1321. |
|rand |	Package rand implements a cryptographically secure pseudorandom number generator. |
|rc4 |	Package rc4 implements RC4 encryption, as defined in Bruce Schneier's Applied Cryptography. |
|rsa |	Package rsa implements RSA encryption as specified in PKCS#1. |
|sha1 |	Package sha1 implements the SHA1 hash algorithm as defined in RFC 3174. |
|sha256 |	Package sha256 implements the SHA224 and SHA256 hash algorithms as defined in FIPS 180-4. |
|sha512 |	Package sha512 implements the SHA-384, SHA-512, SHA-512/224, and SHA-512/256 hash algorithms as defined in FIPS 180-4. |
|subtle |	Package subtle implements functions that are often useful in cryptographic code but require careful thought to use correctly. |
|tls |	Package tls partially implements TLS 1.2, as specified in RFC 5246. |
|x509 |	Package x509 parses X.509-encoded keys and certificates. |
|pkix |	Package pkix contains shared, low level structures used for ASN.1 parsing and serialization of X.509 certificates, CRL and OCSP. |


hash用法：

```
s := "hash this string"
h := sha1.New() // 返回一个hash.Hash变量
// Hash接口： https://golang.org/pkg/hash/#Hash 帮助理解
h.Write([]byte(s))
bs := h.Sum(nil) //Sum(b []byte) appends the current hash to b and returns the resulting slice.
fmt.Println(s)
fmt.Printf("%x\n", bs)  //Use the %x format verb to convert a hash results to a hex string.
```
比php直接调用方法要麻烦一些，理解了倒也还好,比java和C#要方便一些。

## Encoding/编码相关 ##
与编码相关的是encoding包和其子包。比如常用的base64的支持在`encoding/base64`。其用法为:

```
sEncoded := base64.StdEncoding.EncodeToString([]byte("abcdef123'=-=!="))
fmt.Println(sEncoded)
sDecoded := base64.StdEncoding.DecodeString(sEncoded)
fmt.Println(string(sDecoded))
urlEncoded := base64.URLEncoding.EncodeToString([]byte("你好"))
```
从上面可以看到encode和decode 是一对互逆的操作。StdEncoding 和 URLEncoding 的编码结果有一点点区别。

## 待续 ##
*后面将是文件操作，系统相关的内容，应该是最后一节了*

