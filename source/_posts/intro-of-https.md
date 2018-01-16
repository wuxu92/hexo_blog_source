---
title: 了解https通信
date: 2016-03-05 14:29:56
categories:
- network
tags:
- tcp/ip
- https
- network
- security
---

## https和SSL介绍
对于http和https，一般人都不太陌生，随着互联网的发展，这几年越来越的站点开始进行全站https化。

早期只有牵涉到网上购物、网上银行这样的站点才会开启https，现在随着硬件性能的不断提升，全站起用https的开销已经可以接受了，所以最近两年，越来越多的大站开始向全站https过渡。比如前段时间去参加的infoq的一个百度技术分享会，就介绍了百度在全站转向https中的经验；像知乎也开了全站https。一般来说开启https的站点给用户一种放心的感觉，Chrome地址栏那一把绿色的锁在很多人心中就代表“安全”。

https简单地说就是http和ssl/tls的结合，所以https一般看作是**http over ssl, http over tls**或 **http secure**。http协议我们知道是一个不加密的，在计算机网络课程上一般都会进行详细的介绍，三次握手，四次挥手这些都比较熟悉了。但由于https传输的内容是不加密的，在网络上的嗅探、抓包工具抓到的数据可以直接被中间人解析甚至篡改！最典型就是无耻运营商流量劫持，可以说很多的站点趋向于全站https改造就是为了解决无法无天的运营商劫持问题。

ssl协议定义在应用程序协议（http,ftp等）和TCP/IP 协议之间，提供数据安全性分层的机制，SSL为TCP/IP连接提供数据加密、服务器认证、消息完整性验证以及可选的客户机认证。它的主要作用好似提高应用程序之间数据的安全性，对传送的数据进行加密和隐藏，确保数据在传送中不被改变。

### ssl协议
<!-- more -->
ssl协议层对于应用层来说是透明的，我们在编写基于ssl的https应用时，无论是客户端还是服务端都不需要考虑ssl的存在。也就是说对与一般的web开发者（应用层）来说，使用https不需要做比http更多的工作。现在使用的ssl协议为ssl v3.0。

ssl记录协议：ssl记录协议用来封装上层协议（通常指应用层协议，如http）数据的协议，在ssl协议中，所有的数据传输都被封装在该记录中，记录由记录头和长度不为0的记录数据组成。

ssl记录协议定义了要传输数据的格式，它位于一些可靠的数据传输协议之上（如TCP），用于更高层协议的封装（如http）。记录协议主要完成下面的工作：

1. 分组，组合：每个上层应用数据被分成2^14字节或者更小的数据块
2. 压缩、解压缩：压缩是可选的无损压缩，压缩后的内容长度的**增加**不能超多1024字节。
3. 消息认证，在压缩数据上计算消息认证MAC。
4. 加密传输，对压缩数据即MAC进行加密
5. 附加ssl记录头：内容类型，主版本，次版本，压缩长度等字段

关于ssl记录数据包格式的跟多内容可以[查看参考](http://www.cnblogs.com/LittleHann/p/3733469.html)



### tls协议
tls协议是ssl v3的强化版，整个协议上和ssl类似，现在tls协议常用版本为1.2。

tls使用了称为PRF的伪随机函数将密钥扩展成数据块，是更安全的方式。tls与ssl v3.0在计算主密值时采用的方式不同，但都是以客户端和服务器短各自产生的随机数作为输入。tls对填充后的数据长度没有限制，可以防止基于对报文长度进行分析的攻击。

## https通信过程

### ssl握手协议
握手协议是clint和server用ssl连接通信时使用的第一个子协议，ssl握手协议封装在记录协议中，允许服务器与客户机在应用程序传输和接收数据**之前**互相认证，协商加密算法和密钥。该协议主要完成：

1. 客户机认证服务器
2. 协商客户机与服务器，选择双方都支持的密码算法
3. 可选择的服务器认证客户机（一般不执行客户机认证）
4. 使用**公钥加密技术生成共享密钥**
5. 建立ssl连接

ssl握手协议在客户机和服务器之间发送一系列的消息，这些消息都是通过握手协议报文发送，通过类型字段指定使用的ssl握手协议保温类型，常用类型有：

1. hello_request
2. client_hello
3. server_hello
4. certificate
5. server_key_excange
6. certificate_request
7. server_done
8. certificate_verify
9. client_key_exchange
10. finished

client_hello，在此消息中，客户端会产生一个用于生成主密钥的32字节随机数（主密钥由客户端和服务器短的随机数共同生成）；还会将客户端支持的密码套件（CipherSuite）列表根据优先顺序发送给服务器端，每个密码套件都指定了：密码交换算法（RSA, DH, HDE, FORTEZZA等），加密算法（DES, RC4, RC2, 3DES等），认证算法（MD5或者SHA-1），加密方式（流/stream或者分组/block）。

server_hello看作服务器对client_hello的回复，会包含：服务器端采用的ssl版本号（3.0），用于生成主密钥的32字节随机数，服务端采用的用于本次通讯的加密套，以及采用的压缩方法。此消息通知客户端本次通讯采用的方案。

certificate：SSL服务器将"携带自己公钥信息的数字证书"和到根CA整个链发给客户端通过Certificate消息发送给SSL客户端(整个公钥文件都发送过去)，客户端使用这个公钥完成以下任务:

1. 客户端可以使用该公钥来验证服务端的身份，因为只有服务端有对应的私钥能解密它的公钥加密的数据
2. 用于对"premaster secret"进行加密，这个"premaster secret"就是用客户端和服务端生成的Ramdom随机数来生成的，客户端用服务端的公钥对其进行了加密后发送给服务端.(premaster secret就是备选的主密钥)

client_key_exchange：客户端验证服务器证书合法后，利用**证书中的公钥**加密客户端随机生成的“premaster secret”,通过该消息发送给服务器，这一步后，客户端和服务器端都保存了**主密钥**。主密钥会用于ssl通信的数据的加密，是对称加密密钥。

整个握手的流程如图：
![](/images/https/ssl_handshake.png)
注意其中的有阴影部分并不是每次都发送的，它们是可选的或者视情况而定的，比如第二阶段的certificate_request，第三阶段的certificate就是可选的，它们用于验证客户机的身份，这在一般的https通信中并不使用，因为验证客户机身份需要客户机有自己的证书，除非是密级非常高的通信，否则没有这一步骤，下面就是只验证服务器的握手过程：
![](/images/https/ssl_server_cert_only.png)

除了握手协议，还有改变密码协议和ssl警告协议。改变密码协议是为了保障ssl传输过程中的安全性，每隔一段时间改变其加密解密的参数。当一方要该百年其解密参数数，就发送该协议的报文。ssl报警协议用于在通信过程中的一端发现任何异常时像对方发送一条警示消息。

### 数字证书
ssl通信的问题在于确认服务器的身份，黑客可以伪造服务器像客户端发送公钥以和客户机通信。解决这个问题的方法就是上面说的**证书**。

通过**第三方权威机构**颁发的证书（也就是第三方机构用自己的私钥对服务器的公钥进行数字签名），客户端在收到服务器的证书后用第三方机构的公钥对数字签名进行验证，验证通过则可以认为服务器是安全的（是经过认证的正牌服务器）。这里有两个问题：

1. 要有完全可信的第三方机构
2. 第三方权威机构的公钥如何传输到客户机

第一个问题只能认为大家公认的权威机构就是可信的，比如 VeriSign 这种专门颁发证书的机构，如果服务器能经过VeriSign的公钥验证，那么就认为该服务器也是权威的。至于第二个问题，一般操作系统和浏览器在发布的时候就会将权威机构的公钥预置进去，它们本身就会存在于客户机。这也是为什么不要安装第三方打包的操作系统和浏览器，因为这些安装包可能预置了第三方的根证书，这样一些冒牌网站也能通过客户端（浏览器）的认证，导致信息被盗取。

## 在nginx中开启https
在nginx中开启https非常简单，Apache也很方便，这里简单记录一下。

首先在nginx的配置文件中取消对ssl监听的注释，或者增加下面的配置：

```bash
http {
  ssl_session_cache   shared:SSL:10m;
  ssl_session_timeout 10m;
  server {
    listen              443 ssl;
    server_name         www.example.com;
    keepalive_timeout   70;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
  }
}
```
在这个配置中指定了公钥（ssl_certificate）,这个公钥会被发送到通过https连接到此服务器的客户端（浏览器），私钥就是`ssl_certificate_key`指定的文件，一般这个文件的权限**设置为只有nginx主进程可读**，这样可以防止黑客盗取私钥。根据文档，这两个文件可以是同一个文件。

从官方文档给出的配置（`ssl_protocols`配置）可以看出，nginx的默认配置已经不再使用ssl协议了，去年（或者是前年）有人（google?）在网上号召不要再使用ssl，现在对于使用ssl协议的服务器，通过chrome访问时会有安全性提示。

使用下面的命令生产密钥对：

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
```
其中的参数：

- req -x509 用于声明我们要生成一个**自签名**（self-signed）证书，而不是生成一个发送签署证书的请求
- -nodes 因为生产密钥对可以设置一个密码（这在ssh无密码登录中推荐设置），为了nginx能直接读取密钥，不需要这个密码，该选项可以跳过passphrase认证。
- -days 365 指定证书过期的时间，可以自己设置长短
- -newkey rsa:2048 ： 指定同时生成新的key，并指明加密方法为rsa，长度为2048位
- 后面的参数生成的文件的保存目录

该命令会以命令行交互的方式询问该整数的一些信息，比如国家，省市，机构名，联系方式（邮箱），指定的域名等等，这些可以根据自己情况填写。

生产两个文件后，放到nginx配置目录下，也可以是任意目录，只要nginx有读取文件的权限即可。如果不是在nginx的目录下（nginx默认查询目录），需要修改上面的nginx配置文件中，指定密钥文件的配置项。

配置好后重新加载配置或者重启nginx，查看效果。

自签发的证书在浏览器访问时，地址栏的https会有红色的叉子和划线，表示该服务器身份不能通过认证，这是正常的，一般只有第三方权威机构签发的证书才能是绿色的。

参考： 

- [http://zhaoyanblog.com/archives/801.html](http://zhaoyanblog.com/archives/801.html)
- [http://blog.lastww.com/2014/07/09/how-does-https-work/](http://blog.lastww.com/2014/07/09/how-does-https-work/)
- [http://www.cnblogs.com/LittleHann/p/3733469.html](http://www.cnblogs.com/LittleHann/p/3733469.html)
- 白帽子讲Web安全
- [http://nginx.org/en/docs/http/configuring_https_servers.html](http://nginx.org/en/docs/http/configuring_https_servers.html)
- [https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04)