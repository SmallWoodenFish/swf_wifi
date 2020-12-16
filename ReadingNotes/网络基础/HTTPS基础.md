# HTTPS基础


<!-- TOC -->

- [HTTPS基础](#https基础)
    - [1. HTTPS通信主要过程](#1-https通信主要过程)
    - [2. Keyless SSL](#2-keyless-ssl)

<!-- /TOC -->

----


参考文档：

[CA证书（数字证书的原理）](https://cloud.tencent.com/developer/article/1559900)：HTTPS方案的形成，每个环节所带来的新问题与新应对

[Announcing Keyless SSL™: All the Benefits of CloudFlare Without Having to Turn Over Your Private SSL Keys](https://blog.cloudflare.com/announcing-keyless-ssl-all-the-benefits-of-cloudflare-without-having-to-turn-over-your-private-ssl-keys/)：CloudFlare发布Keyless SSL方案

[Keyless SSL: The Nitty Gritty Technical Details](https://blog.cloudflare.com/keyless-ssl-the-nitty-gritty-technical-details/)：CloudFlare发布的Keyless SSL方案细节

[Keyless SSL的技术细节](https://blog.csdn.net/xuyongjiande/article/details/80403541)：上文的中文版

----

### 1. HTTPS通信主要过程

Step 1: Client向server发送请求

---- say hello ----

Step 2: Server向client 发送 自己的数字证书。数字证书包括server的公钥，以及颁发机构信息和指纹。

Step 3: Client收到证书后，会向颁发机构校验证书真实性，并校验颁发机构的证书，直到迭代到“已信任的证书”（操作系统内置 or 用户手动添加），通过校验。

---- 完成证书校验 ----

Step 4: Client生成一个随机密钥premaster secret，发给server，让server自证身份。

Step 5: Server拿到premaster secret，用私钥加密，发回给client。

Step 6: Client收到私钥加密后的premaster secret，用公钥解密，如果解密后和原始premaster secret相同，证明server是可信的

---- 完成服务器身份校验 ----

Step 7: client生成一个对称加密算法和密钥secret，使用公钥加密后发给server

Step 8: server用私钥解密拿到加密算法和密钥secret

---- 完成密钥协商，开始通信 ----

Step 9: client和server使用这组加密算法和secret进行通信


### 2. Keyless SSL

一些HTTPS相关的云服务需要由云厂商直接和用户建立HTTPS连接，例如CDN、抗DDoS高防等，而部分业务敏感的企业（如金融业）不能将SSL证书托管到外部机构。因此需要找到无需SSL私钥就能建立HTTPS连接的方法。

核心思想就是改变上述步骤的Step 5。

用户与云厂商直接进行通信时，server由企业变为云厂商，而云厂商没有私钥，step 5无法正常进行。不过，可以在企业内架设一台专用的机器key server，帮助云厂商处理premaster secret（无论是给premaster secret私钥加密，还是将公钥加密后的premaster secret私钥解密），并且直接把结果返回给云厂商server。

在整个通信过程中，只有这一步需要直接使用私钥，其他功能均可由云厂商server代理。

不过，必须要控制只有云厂商server可以请求企业架设的key server，通常使用TLS双向认证。

