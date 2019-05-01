翻了下 Let’s Encrypt 的官方博客，其中有一篇提到了2019年的一些计划，觉得很有趣，特此分享给大家。

Let’s Encrypt 是世界上最大、最流行、且真正免费的 CA 机构，他们对于 HTTPS 普及功不可没。部署 HTTPS 需要购买 CA 证书，CA 机构是追求盈利的机构，不言而喻 Let’s Encrypt 对他们的冲击有多大，而对网站拥有者来说，这就是福音了，免费、免费、真正的免费，而且其功能丝毫不比商业 CA 机构差，而且更透明。

我写过不少关于 Let’s Encrypt 的文章，也贡献过力量（见 https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au 仓库），大家可以参考我以前写的文章。**经常在互联网上看到很多人推销收费证书，有的时候还是想大声疾呼，请使用免费证书！！**

在2019年的展望中，Let’s Encrypt 从四个方面做了描述，分别是增长、新特性、基础设施、财务状况。

### 增长 

从整个大环境上看，根据 Mozilla 的统计，2018年加密的网页从67%上涨到77%。从 Let’s Encrypt 的角度看，它的增长趋势见下图：

![newplot.png](newplot.png) 

在 2019 年，希望激活的证书达到1.2亿，fully qualified domains（主机名+注册域）激活的证书希望是 2.15 亿（这二个数字不一致的原因在于 SAN 证书的原因）。

Let’s Encrypt 开创性的提出了 ACME 协议，让证书管理（申请、续期、撤销）完全自动化，目前 Let’s Encrypt 已经有了 85 个客户端软件支持了。

Apache 已经直接支持 Let’s Encrypt，2019年希望 Nginx 官方也能以模块的方式直接支持。

### 新特性 

在 2018 年，最重要的新特性就是 ACMEv2 协议的发布，另外也支持通配符证书了（这个是大杀器，在功能上和传统CA没有任何区别了）。

在 2019 年的规划上，主要有三个。

（1）在 2018 年，Let’s Encrypt 支持签发 ECDSA 服务器证书，在 2019年，其中间证书和根证书也讲支持 ECDSA 签名，从而构建完整的 ECDSA 证书链。

（2）Let’s Encrypt 在签发证书的时候，会向第三方 CT log 服务器提交日志，但他们认为第三方的服务不够稳定，所以在 2019 年会自己构建 Certificate Transparency (CT) log 服务，越来越成熟的 CA 机构。

（3）构建多条校验机制，在用户申请证书的时候，Let’s Encrypt一般使用校验域名授权的方式校验用户的身份，但有潜在的安全风险，所以在2019年他们校验的时候会通过多种机制确保申请者的身份，个人觉得这是非常棒的一个特性，不像传统 CA 机构，由于无法人工审核证书申请用户身份，所以这个特性能够避免很多安全风险。

### 基础设施 

不要小看 Let’s Encrypt 的基础设施，每天能够发布数百万张证书，每天要接收 4000万次 OCSP 查询，所以需要很好的服务能力。

他们拥有 45 个机架，其中数据库的容量增长是比较大的。有专业的 SRE 团队，共六个员工，相当高效的一个工程团队。

### 财务 

在 2019 年 facebook一次性给了 Let’s Encrypt 三年的白金赞助，当然思科、Google、Mozilla 等大型企业早就赞助他们了。

他们今年的预算是 $3.6M，可见团队和运作模式非常高效。

而且在 2019年，他们新弄了一个网站，Internet Security Research Group (ISRG)，也就是说未来 ISRG 不仅仅有 Let’s Encrypt 一个项目，他们还有更多的事业去搞，以后应该换种称呼 Let’s Encrypt，这就是 ISRG / Let’s Encrypt。

--- 

相关文章：

- [Certobot管理Let's Encrypt证书的几个经验](https://mp.weixin.qq.com/s/hKvtDDQw7EHSGFRGT4QVbw) 
- [let's encrypt通配符证书自动续期工具支持腾讯云DNS了](https://mp.weixin.qq.com/s/uZe3z-8s2zrqxrcvcfV83g) 
- [主流浏览器直接信任Let’s Encrypt根证书，宣告它成为顶级CA](https://mp.weixin.qq.com/s/jFkfe7O2sPCyXFcFzpi1Qw) 
- [不会自动为Let’s Encrypt通配符证书续期？我写了个小工具](https://mp.weixin.qq.com/s/aTjl79NsE6WkS47RGlX_gg)  
- [Let’s Encrypt证书支持CT，让你的网站更安全](https://mp.weixin.qq.com/s/Z-dNokZFbfOmoKFlOlZLWA) 
- [Let's Encrypt 终于支持通配符证书了](https://mp.weixin.qq.com/s/Y-_0lLhZhY3IlPM--d3PQg) 

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），一直在用心写。