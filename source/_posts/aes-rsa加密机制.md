---
title: AES-RSA加密机制
date: 2017-12-05 11:37:00
tags: [RSA,AES]
categories: 加解密
---

在服务器与终端设备进行HTTP通讯时，常常会被网络抓包、反编译（Android [APK反编译工具](https://link.jianshu.com?t=http://idoog.me/?p=2933)）等技术得到HTTP通讯接口地址和参数。为了确保信息的安全，我们采用AES+RSA组合的方式进行接口参数加密和解密。

1.关于RSA加密机制：公钥用于对数据进行加密，私钥对数据进行解密，两者不可逆。公钥和私钥是同时生成的，一一对应。比如：A拥有公钥，B拥有公钥和私钥。A将数据通过公钥进行加密后，发送密文给B，B可以通过私钥和公钥进行解密。

2.AES加密也叫对称加密：A用密码对数据进行AES加密后，B用同样的密码对密文进行AES解密。


具体操作方法：

1.在终端中采用openssl方式输入密钥的相关属性（公司名、邮箱等），然后在终端当前所在的地址下，生成公钥和私钥共7个文件（7个文件如何使用请看附录的拓展了链接）。

2.此时假设Android客户端拥有公钥PublicKey,服务器端拥有公钥PublicKey和私钥PrivateKey。

3.安卓发送请求到服务器端：安卓随机生成Byte[]随机密码，假设RandomKey=“123456”，通过AES算法，对Json数据利用进行加密。

4.但是此刻服务器并不知道客户端的RandomKey是什么，因此需要同时将Randomkey传给服务器，否则服务器无法通过AES对Json数据进行解密。但是如果直接发送请求，Randomkey就会暴露，所以要对RandomKey进行不可逆的RSA加密。

5.安卓将使用Randomkey进行AES加密的Json数据，和使用PublicKey进行RSA加密的RandomKey通过HTTP传送到服务器端。数据请求工作完成。

6.服务器端接收到AES加密的Json数据和Rsa加密的RandomKey数据。

7.服务器通过私钥PrivateKey对加密后的RandomKey进行Rsa解密。得到安卓生成的原始Randomkey。

8.利用原始的RandomKey对加密后的Json数据进行AES对称解密。至此已经得到安卓端发过来的原始Json数据。进行常规的服务器业务操作，然后将返回数据通过安卓端的RandomKey进行AES加密gouhou后，Response返回。

9.安卓端接收到Response的数据后，利用之前本地生成的RandomKey直接进行AES解密即可。

详细的流程图可以查看下图。

![image](http://upload-images.jianshu.io/upload_images/265023-b64e781d365f1200.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

注意事项：

1.在实际的开发过程中，发现RSA和AES有不同的密文生成标准，会不兼容IOS。IOS在RSA算法中需要的公钥与JAVA不同。详细的解决方案请查看：http://www.cnblogs.com/makemelike/articles/3802518.html

2.AES加密不可以使用超过128Byte的KEY，因为在jdk1.7以上的版本不支持超过128Byte的KEY。

小结：从性能上来测，整个客户端送加密数据开始到解密得到回传的原始数据不超过300ms（Iphone4和Centos Java服务器传输测试）。本方案没有采用TOKEN的方式，或许以后用到。公钥如何更新也需要继续完善。

附：具体的JAVA和IOS加密解密Demo迟点整理给出。