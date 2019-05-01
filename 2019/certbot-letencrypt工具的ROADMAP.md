今天有个开发者在 certbot-letencrypt-wildcardcertificates-alydns-au 仓库提了一个非常好的 issues，临时解决了下，也顺便想了想这个工具的后续 ROADMAP，将相关的想法写成本文。

certbot-letencrypt-wildcardcertificates-alydns-au 这个工具（https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au），主要作用是自动生成/renew（续期）letencrypt 通配符证书，关于这个工具我写过一篇文章[《不会自动为Let’s Encrypt通配符证书续期？我写了个小工具》](https://mp.weixin.qq.com/s/aTjl79NsE6WkS47RGlX_gg) ，可以看一看。

这个工具目前已经 45个fork，157个star，给了我很大的鼓舞，很多开发者也发现了很多我没有考虑到的问题，感觉收获很大。工具虽然很小，但如果你想使用免费的 ssl 证书部署 https 服务，那么非常有用。

所以说，即使你写的工具或代码很小、很简单，但如果解决了实际问题，仍然有很大的价值。

1：今天遇到了什么问题

提的 issues 主要说，如果想为 example.com 和 *.example.com 生成一张 SAN 证书，运行该工具的时候会失败，比如：

```
$ ./certbot-auto certonly  -d example.com -d *.example.com  --manual --preferred-challenges dns  --dry-run --manual-auth-hook /脚本目录/au.sh
```

原因在哪儿呢？example.com 和 *.example.com 对应的 TXT 记录名是同一个，工具在运行的时候会调用二次au.sh，该 sh 会调用 alydns.php（自动生成DNS TXT记录）接口，alydns.php 的逻辑是删除同名的 TXT 记录，然后 add 相应的 TXT 记录。

聪明的读者可能想到了，certbot 在验证 DNS 记录的时候，只会获取到一条 TXT 记录（实际上应该有二条），这样校验失败，certbot renew 更新失败。

具体的问题和临时解决方案见 [https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au/issues/21](https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au/issues/21)。

大概的原理就是 alydns.php 在运行的时候不删除相应的 TXT 记录，DNS 能够增加多条同名的 TXT 记录（这是我今天的收获）。

这个临时解决方案有个最大的弊端就是，由于 TXT 记录永远不删除，所以会越来越多，可能会超过 DNS 条目数的上限。

其实 certbot 提供了一个 hook（--manual-cleanup-hook），可以在执行成功后，进行一些 cleanup 工作（比如删除 DNS TXT 记录），比如：

```
$ ./certbot-auto certonly  -d example.com -d *.example.com  --manual --preferred-challenges dns  --dry-run --manual-auth-hook /脚本目录/au.sh --manual-cleanup-hook /脚本目录/cleanup.sh
```

但这样我就要为这个工具添加多个 php 文件（用于删除 DNS TXT 记录），有人说，你不能合并到 alydns.php 文件中吗？可惜的是 certbot 的 hook 不支持为 hook 脚本添加命令行参数。

如果添加 —manual-cleanup-hook，这个工具就庞大了，用户会望而生畏，所以终极解决方案我再思考下。

2：同步和部署证书

这个工具只是生成/更新证书，一般情况下，更新证书后，需要重新启动 web 服务器，以便加载新的证书文件。

如果运行 certbot 的机器和 web 服务器是同一台，很好解决：

```
certbot-auto renew --manual --preferred-challenges dns -deploy-hook  "service nginx restart"  --manual-auth-hook /脚本目录/au.sh 
```

可很多公司的服务器肯定不止一台，一般使用专用的 certbot 机器生成/更新证书，然后将证书同步到相关的web服务器上，应用场景有很多，很难概括。

今天我查看了 certbot 的官方文档，有了新的想法，后续打算写个工具解决下，虽然不一定完美，但有很多开发者一起协助，相信会越来越好。

最后，今天是我春节前最后一天工作，能够解决一个问题，非常开心，祝大家新春快乐，多多转发我的公众号文章，相信你不会失望的。

certbot 和 letencrypt 相关文章：

- [Certobot管理Let's Encrypt证书的几个经验](https://mp.weixin.qq.com/s/hKvtDDQw7EHSGFRGT4QVbw) 
- [let's encrypt通配符证书自动续期工具支持腾讯云DNS了](https://mp.weixin.qq.com/s/uZe3z-8s2zrqxrcvcfV83g) 
- [主流浏览器直接信任Let’s Encrypt根证书，宣告它成为顶级CA](https://mp.weixin.qq.com/s/jFkfe7O2sPCyXFcFzpi1Qw) 
- [不会自动为Let’s Encrypt通配符证书续期？我写了个小工具](https://mp.weixin.qq.com/s/aTjl79NsE6WkS47RGlX_gg)  
- [Let’s Encrypt证书支持CT，让你的网站更安全](https://mp.weixin.qq.com/s/Z-dNokZFbfOmoKFlOlZLWA) 
- [Let's Encrypt 终于支持通配符证书了](https://mp.weixin.qq.com/s/Y-_0lLhZhY3IlPM--d3PQg) 

--- 

欢迎大家关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳）。