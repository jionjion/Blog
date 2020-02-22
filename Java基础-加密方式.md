---
title: Java基础-加密方式
abbrlink: a355509b
date: 2020-01-30 09:43:44
categories:
  - Java
  - Code
tags: [Java, Encode, Decode]
---



# 简介

介绍了JAVA中常用到的加解密方式,对信息安全传递进行了简要介绍.



## 相关知识
**明文:** 等待加密的信息
**密文:** 经过加密后的明文
**加密:** 明文转为密文的过程
**加密算法:** 明文转为密文的转换算法
**加密密钥:** 通过加密算法进行加密操作的密钥
**解密:** 将密文转为明文的过程
**解密算法:** 密文转为明文的算法
**解密密钥:** 通过解密算法进行解密操作的密钥
**密码体制:** 明文空间,密文空间,秘钥空间,加密算法和解密算法五部分构成
**密码协议:** 安全协议,以密码学为基础的信息交换的通信协议,目的是在网络环境中提供安全的服务.
**柯克霍夫原则:** 数据的安全基于秘钥而不是算法的保密.系统的安全取决于秘钥,对秘钥保密,对算法公开.



## 实现方式
通过扩展JDK的配置文件,指明包的位置实现加载,配置文件在JDK的安装目录下的`jre\lib\security`的`java.security`文件中,该文件中加入了JAVA自带的实现方式.
通过添加新的实现方式,完成注入.
``` 
security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=sun.security.ec.SunEC
```

也可以通过`addProvider()`或`insertProviderAt()`实现动态添加算法实现类

**常用的加密方式**
`java.security`消息摘要,安全框架实现的基础
`javax.crypto`安全消息摘要,消息认证码,最为完整的加解密方式
`java.net.ssl`安全套接字,网络传输时加密.

**第三方扩展**
*Bouncy Castle*
支持以上两种配置方式
*Commons Codec*
Base64,二进制,十六进制,字符集编码,URL编码




# 安全加解密的方式



## Base64算法

使用Base64算法,完成基础的敏感字段的加解密方式.



## 消息摘要算法

消息摘要算法是一种单向的加密方式,常用来验证数据完整性和作为数字签名,通过对此明文加密后与密文的重合度,进而判断数据真实性.



### MD消息摘要算法

| 算法 | 摘要长度 | 实现方        |
| ---- | -------- | ------------- |
| MD2  | 128      | JDK           |
| MD4  | 128      | Bouncy Castle |
| MD5  | 128      | JDK           |


![消息摘要加密][1]



### SHA消息摘要算法

安全散列算法,生成的摘要信息为固定长度

| 算法    | 摘要长度 | 实现方        |
| ------- | -------- | ------------- |
| SHA-1   | 160      | JDK           |
| SHA-224 | 224      | Bouncy Castle |
| SHA-256 | 256      | JDK           |
| SHA-384 | 384      | JDK           |
| SHA-512 | 512      | JDK           |


![SHA消息摘要算法][2]



### MAC消息摘要算法

散列函数算法

| 算法       | 摘要长度 | 实现方        |
| ---------- | -------- | ------------- |
| HmacMD2    | 128      | Bouncy Castle |
| HmacMD4    | 128      | Bouncy Castle |
| HmacMD5    | 128      | JDK           |
| HmacSHA1   | 160      | JDK           |
| HmacSHA224 | 224      | Bouncy Castle |
| HmacSHA256 | 256      | JDK           |
| HmacSHA384 | 384      | JDK           |
| HmacSHA512 | 512      | JDK           |


![MAC消息摘要算法][3]



## 对称加密算法



### DES对称加密算法

数据加密方式标准

| 秘钥长度 | 默认 | 工作模式                                        | 填充方式                                                     | 实现方        |
| -------- | ---- | ----------------------------------------------- | ------------------------------------------------------------ | ------------- |
| 56       | 56   | ECB,CBC,PCBC,CTR,CTS,CFB, CFB8~128,OFB,OFB8~128 | NoPadding,PKCS5Padding,ISO10126Padding                       | JDK           |
| 64       | 56   | 同上                                            | PKCS7Padding,ISO10126d2Padding,X932Padding,ISO7816dPadding, ZeroBytePadding | Bouncy Castle |

![DES对称加密算法][4]



### 3DES对称加密算法

对DES加密算法的进步一升级.

| 秘钥长度 | 默认 | 工作方式                                       | 填充方式                                                     | 实现方 |
| -------- | ---- | ---------------------------------------------- | ------------------------------------------------------------ | ------ |
| 112,168  | 168  | ECB,CBC,PCBC,CTR,CTS,CFB,CFB8~128,OFB,OFB8~128 | NoPadding,PKCS5Padding,ISO10126Padding                       | JDK    |
| 128,192  | 168  | 同上                                           | PKCS7Padding,ISO10126d2Padding,X932Padding,ISO7816dPadding, ZeroBytePadding | BC     |



### AES对称加密算法

高级加密标准

| 秘钥长度    | 默认 | 工作方式                                       | 填充方式                               | 实现方        |
| ----------- | ---- | ---------------------------------------------- | -------------------------------------- | ------------- |
| 128,192,256 | 128  | ECB,CBC,PCBC,CTR,CTS,CFB,CFB8~128,OFB,OFB8~128 | NoPadding,PKCS5Padding,ISO10126Padding | JDK           |
| 同上        | 同上 | 同上                                           | PKCS7Padding,ZeroBytePadding           | Bouncy Castle |

![AES对称加密算法][5]



### PBE对称加密算法

基于口令加密对已有算法的包装

| 算法                            | 秘钥长度       | 默认 | 工作模式 | 填充方式                                                  | 实现          |
| ------------------------------- | -------------- | ---- | -------- | --------------------------------------------------------- | ------------- |
| PBEWithMD5AndDES                | 64             | 64   | CBC      | PKCS5Padding,PKCS7Padding,ISO10126Padding,ZeroBytePadding | Bouncy Castle |
| PBEWithMD5AndRC2                | 112            | 128  |          |                                                           |               |
| PBEWithSHA1AndDES               | 64             | 64   |          |                                                           |               |
| PBEWithSHA1AndRC2               | 128            | 128  |          |                                                           |               |
| PBEWithSHAAndIDEA-CBC           | 128            | 128  |          |                                                           |               |
| PBEWithSHAAnd2-KeyTripleDES-CBC | 128            | 128  |          |                                                           |               |
| PBEWithSHAAnd3-KeyTripleDES-CBC | 192            | 192  |          |                                                           |               |
| PBEWithMD5AndDES                | 56             | 56   | CBC      | PKCS5Padding                                              | JDK           |
| PBEWithMD5AndTripleDES          | 112,168        | 168  |          |                                                           |               |
| PBEWithSHA1AndDESede            | 112,168        | 168  |          |                                                           |               |
| PBEWithSHA1AndRC2_40            | 40~1024(8倍数) | 128  |          |                                                           |               |


![PBE对称加密算法][6]



## 非对称加密算法

使用公钥和私钥完成对数据的加密过程.

1. 初始化发送方秘钥
	- KeyPairGenerator  生成秘钥对
	- KeyPair  生成秘钥
	- PublicKey  公钥
2. 初始化接收方秘钥
	- KeyFactory  生成公钥私钥
	- X509EncodedKeySpec  秘钥编码
	- DHPublicKey  公钥
	- DHParameterSpec  参数空间
	- KeyPairGenerator 秘钥对
	- PrivateKey  私钥
3. 秘钥构建
	- KeyAgreement            提供协议
	- SecretKey  命名秘钥
	- KeyFactory      创建秘钥
	- X509EncodedKeySpec  秘钥编码
	- PublicKey   公钥
4. 加解密



### DH非对称加密算法

秘钥交换算法

| 秘钥长度         | 默认 | 工作模式 | 填充方式 | 实现方 |
| ---------------- | ---- | -------- | -------- | ------ |
| 512~1024(64倍数) | 1024 | 无       | 无       | JDK    |



### RSA非对称加密算法

因子分解非对称加密算法

| 秘钥长度            | 默认 | 工作模式 | 填充方式                                                     | 实现方        |
| ------------------- | ---- | -------- | ------------------------------------------------------------ | ------------- |
| 512~65536(64整数倍) | 1024 | ECB      | NoPadding,PKCS1Padding,OAEPWithMD5AndMGF1Padding,OAEPWithSHA1AndMGF1Padding,OAEPWithSHA256AndMGF1Padding,OAEPWithSHA384AndMGF1Padding,OAEPWithSHA512AndMGF1Padding | JDK           |
|                     | 2048 | NONE     | NoPadding,PKCS1Padding,OAEPWithMD5AndMGF1Padding,OAEPWithSHA1AndMGF1Padding,OAEPWithSHA256AndMGF1Padding,OAEPWithSHA384AndMGF1Padding,OAEPWithSHA512AndMGF1Padding | Bouncy Castle |

![RSA非对称加密算法][7]



### ElGamal非对称加密算法

基于离散对数的公钥加密算法

| 秘钥长度         | 默认 | 工作方式 | 填充方式                                                     | 实现方        |
| ---------------- | ---- | -------- | ------------------------------------------------------------ | ------------- |
| 160~16384(8倍数) | 1024 | ECB,NONE | NoPadding,PKCS1Padding,OAEPWithMD5AndMGF1Padding,OAEPWithSHA1AndMGF1Padding,OAEPWithSHA256AndMGF1Padding,OAEPWithSHA384AndMGF1Padding,OAEPWithSHA512AndMGF1Padding | Bouncy Castle |

![EIGamal非对称加密算法][8]



## 数字签名算法



### RSA签名算法

| 算法             | 秘钥长度          | 默认长度 | 签名长度       | 实现方        |
| ---------------- | ----------------- | -------- | -------------- | ------------- |
| MD2WithRSA       | 512~65536(64倍数) | 1024     | 与秘钥长度相同 | JDK           |
| MD5WithRSA       |                   |          |                |               |
| SHA1WithRSA      |                   |          |                |               |
| SHA224WithRSA    | 512~65536(64倍数) | 2048     | 与秘钥长度相同 | Bouncy Castle |
| SHA256WithRSA    |                   |          |                |               |
| SHA384WithRSA    |                   |          |                |               |
| SHA512WithRSA    |                   |          |                |               |
| PIPEMD128WithRSA |                   |          |                |               |
| PIPEMD160WithRSA |                   |          |                |               |

![RSA数字签名算法][9]



### DSA签名算法

数字签名算法

| 算法          | 秘钥长度         | 默认长度 | 实现方        |
| ------------- | ---------------- | -------- | ------------- |
| SHA1WithDSA   | 512~1024(64倍数) | 1024     | JDK           |
| SHA224WithDSA | 512~1024(64倍数) | 1024     | Bouncy Castle |
| SHA256WithDSA | 512~1024(64倍数) | 1024     | Bouncy Castle |
| SHA384WithDSA | 512~1024(64倍数) | 1024     | Bouncy Castle |
| SHA512WithDSA | 512~1024(64倍数) | 1024     | Bouncy Castle |

![DSA数字签名算法][9]



### ECDSA签名算法

椭圆曲线数字签名算法

| 算法               | 秘钥长度 | 默认 | 签名长度 | 实现方            |
| ------------------ | -------- | ---- | -------- | ----------------- |
| NONEWithECDSA      | 112~571  | 256  | 128      | JDK/Bouncy Castle |
| PIPEMD160WithECDSA | 112~571  | 256  | 160      | Bouncy Castle     |
| SHA1WithECDSA      | 112~571  | 256  | 160      | JDK/Bouncy Castle |
| SHA224WithECDSA    | 112~571  | 256  | 224      | Bouncy Castle     |
| SHA256WithECDSA    | 112~571  | 256  | 256      | JDK/Bouncy Castle |
| SHA386WithECDSA    | 112~571  | 256  | 384      | JDK/Bouncy Castle |
| SHA512WithECDSA    | 112~571  | 256  | 512      | JDK/Bouncy Castle |

![ECDSA数字签名算法][9]





## 示例代码

- [*aesDemo*][10]													  AES对称加密算法
- [*base64Demo*][11]												Base64算法
- [*des3Demo*][12]												    3DES对称加密算法
- [*desDemo*][13]													  DES对称加密算法
- [*dhDemo*][14]													   DH非对称加密算法
- [*dsaDemo*][15]												     DSA数字签名算法
- [*ecdsaDemo*][16]										     	ECDSA数字签名算法
- [*elGamalDemo*][17]											EIGaml非对称加密算法
- [*macDemo*][18]												   MAC消息摘要算法
- [*mdDemo*][19]													MD消息摘要算法
- [*pbeDemo*][20]												   PBE对称加密算法
- [*rsaDemo*][21]											    	RSA非对称加密算法
- [*rsaDemoTwo*][22]							 				RSA数字签名算法
- [*shaDemo*][23]							   					SHA消息摘要算法





[1]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-01.png "encryption-01"
[2]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-02.png
[3]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-03.png "encryption-03"
[4]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-04.png "encryption-04"
[5]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-05.png "encryption-05"
[6]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-06.png "encryption-06"
[7]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-07.png "encryption-07"
[8]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-08.png "encryption-08"
[9]: https://www.github.com/jionjion/Picture_Space/raw/master/WorkSpace/Java/javaBase/encryption-09.png "encryption-09"



[10]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/aesDemo
[11]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/base64Demo
[12]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/des3Demo
[13]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/desDemo
[14]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/dhDemo
[15]:https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/dsaDemo
[16]:https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/ecdsaDemo
[17]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/elGamalDemo
[18]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/macDemo
[19]: https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/mdDemo
[20]:https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/pbeDemo
[21]:https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/rsaDemo
[22]:https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/rsaDemoTwo
[23]:https://www.github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/javaEncryption/shaDemo

