在前面两篇文章中，对用户口令进行加密的方式其实称为 Password-based encryption (PBE)，算法实现很简单，那是不是有更好和更标准的 PBE 实现呢？

基于 Hash+salt 的算法最大的问题在于 Hash 函数的运算太快了，虽然加盐让暴力攻击和彩虹表攻击的可行性大大减低，但现在攻击者能在非常快速的硬件（包括 GPU）上运行，如果**时间足够**，还是有很大几率完成暴力破解。

那有没有更好的解决方案吗？如果让口令运算运算的慢一点，那么攻击者破解的速度也将上不去，这样是否就能更好的保护明文口令？

在密码学中，key derivation function (KDF) 函数非常重要，它可以通过一个 master key（在 HTTPS 中用的非常多）、口令（password）、passphrase（密码学随机数生成器）生成一个或多个强壮的密钥，这些密钥本身被密码学算法使用（比如 AES、RSA 等等）。

用户的口令通过 PBF（前两篇文章讲解的算法实现）生成的口令密文不是密钥，所以最终结果不是用于密码学用途，是为了避免口令被破解，但本质上 BPF 算法也可以通过 KDF 函数实现，也就是利用 KDF 函数生成口令密文。

KDF 同样基于 Hash 函数，也有 salt 机制，当然最重要的是有**迭代因子**这个概念，有了迭代因子，会让处理速度变慢，减少爆破风险。

KDF 主要有三种实现，分别是**PBKDF2、bcrypt、scrypt**，这篇文章主要讨论 PBKDF2。

稍微休息下，希望大家理解上述概念之间的区别。

KDF 本质上属于 Key stretching、key strengthening，如果你了解 HTTPS，那么可能比较熟悉，比如在握手阶段，HTTPS 将 Premaster Secret 和客户端服务器端的随机数导出为 Master Secret，然后再将 Master Secret 导出为多个密钥块，这些密钥块包含 AES 的加密密钥或者初始化向量，用户后续通信数据的加密和完整性保护。

PBE 算法标准定义在 RFC 2898 文档中，大概的公式如下：

```
DK = PBKDF2(PRF, Password, Salt, c, dkLen)
```

- PRF 是一个伪随机函数，可以简单的理解为 Hash 函数。
- Password 表示口令 。 
- Salt 表示盐值，一个随机数。
- c 表示迭代次数。
- dkLen 表示最后输出的密钥长度。 

如果 c 的数值越大，那么运算速度就越慢，增加了时间复杂度，攻击者破解的成功率就会下降。

使用 PHP 语言说明 PBKDF2 函数的使用：

```
$password = '明文口令';
// 随机的盐值
$salt = openssl_random_pseudo_bytes(12);
$keyLength = 40;
$iterations = 10000;
$generated_key = openssl_pbkdf2($password, $salt, $keyLength, $iterations, 'sha256');
//转换为 16 进制
echo bin2hex($generated_key)."\n";
```

对这个过程循环2000次，总共需要**16秒**，而如果运行简单的 Hash+salt，循环2000次，运行时间不到**0.1秒**。 

从这个角度看，建议大家使用 PBKDF2 保护你的口令，但现在业界的保护口令的标准算法是 bcrypt ，下一篇文章会讲解。

口令保护系列文章：

- [如何安全存储口令？了解下Hash加盐的原理](https://mp.weixin.qq.com/s/1X_aOoadk1CCBzGFC-EM3w)
- [一个可实践的「Hash加盐」技术方案](https://mp.weixin.qq.com/s/WZQ4aJLnnSnKym5yBp3v2g)

--- 

了解我的书[《深入浅出HTTPS：从原理的实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)，如果觉得还不错，还请在豆瓣上做个评论（地址：https://book.douban.com/subject/30250772/，或点击“原文连接”）。

 