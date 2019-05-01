最后谈一谈安全保护口令的的标准算法，这就是 bcrypt 算法。为了把事情说清楚，分两篇文章描述：

- 说说 bcrypt 算法，以及通过PHP的crypt()算法进一步参数阐述，虽然crypt()算法已经不推荐使用了，但对于理解 bcrypt 算法还是非常好的，可以看看它的历史。
- 在一个系统中，如果有多种开发语言，那么 bcrypt 算法是否通用呢？通过 PHP 和 Ptyhon 语言进行描述。

本文主要理解 bcrypt 算法，bcrypt 算法可以认为是 KDF 函数的一种实现，也有迭代因子的概念。

bcrypt 算法基于 Blowfish 块密钥算法，bcrypt 算法已经有10多年的历史，而 Blowfish 密钥算法更是有20多年的历史，久经考验，所以被认为是 Hash 加密口令的标准算法。

bcrypt 算法在内部会使用内存初始化 hash 过程，由于需要内存，虽然在 CPU 上运行很快，但在 GPU 并行运算却不快，这也减缓了攻击者的破解速度。

接下去我使用 PHP 语言中的 crypt() 函数介绍如何使用 bcrypt 算法，如果你对 Hash 保护口令了解的不多，那么使用 crypt() 函数可能会存在很多问题。

首先必须明确 crypt() 函数并不是 bcrypt 算法，它可以基于**多种不同的 Hash 算法**。

该函数的原型：

```
string crypt ( string $str [, string $salt ] )
```

看上去很简单，但隐藏了很多内容。

如果你仅仅调用 crypt()，会根据操作系统版本和 PHP 版本使用相应的 Hash 算法，而且如果不显示的输入 salt，可能会得到一个弱 salt，所以不推荐这样调用 crypt() 函数，因为**屏蔽**了很多细节。

那么如何选定 bcrypt 算法（Blowfish）、迭代因子、salt，先看一个例子：


```
if (CRYPT_BLOWFISH == 1) {
    echo 'Blowfish:' . crypt('abcde', '$2a$07$woshiyigesaltzhi') . "\n";
}
```

- $2a$ 表示使用 bcrypt 算法（注意：如果是 PHP 5.3.7 后续版本，使用 $2y$，修复了安全风险）。
- 07 表示迭代次数为 7 次。
- 最后一个 $ 后面内容为 salt 值。

接下去看看上面代码的输出：

```
Blowfish:$2a$07$woshiyigesaltzhi$$$$$.lrU488y7E1Xw.JA4uizIu.PBSSe7t4y
```

也就是说返回值包含了 crypt() 函数相关信息，比如告诉你使用了 bcrypt 算法，迭代因子是 7，salt 是 woshiyigesaltzhi$$$$$，剩下的部分就是口令密文。

此处，遗留一个问题，crypt() 运算出来的口令密文包含 salt 是否不安全？**这会在下一篇中描述**。
 
crypt() 也可以使用其它的 Hash 函数，比如：

- CRYPT_MD5： $1$
- CRYPT_BLOWFISH：$2a$ 或 $2y$
- CRYPT_SHA256：$5$
- CRYPT_SHA512：$6$

大家大概明白 crpyt() 函数的使用了，可能使用的时候有点麻烦，所以建议包装下该函数：

```
function better_crypt($input, $rounds = 7) {
	$salt = openssl_random_pseudo_bytes(22);
	return crypt($input, sprintf('$2a$%02d$', $rounds) . $salt);
}
```

不管怎么说，crypt() 函数完全基于底层的 C 函数，运行环境也依赖于操作系统和PHP版本，系统和代码迁移的时候可能有多种问题，所以 PHP 从 5.5 版本以后建议使用 **Password Hashing Functions**，这在下一篇会详细说一下。

最后简单提下 scrypt 算法，它是一种新的口令保护算法，是另外一种思路，被认为是口令保护的业界标准算法，但由于时间较短，现在建议还是使用 **bcrypt() 算法**。

口令保护系列文章：

- [如何安全存储口令？了解下Hash加盐的原理](https://mp.weixin.qq.com/s/1X_aOoadk1CCBzGFC-EM3w)
- [一个可实践的「Hash加盐」技术方案](https://mp.weixin.qq.com/s/WZQ4aJLnnSnKym5yBp3v2g)
- [PBKDF2函数，比「Hash加盐」更好的口令保护方案](https://mp.weixin.qq.com/s/16Rze8cPFfrHolBMRhznmA)

--- 

了解我的书[《深入浅出HTTPS：从原理的实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)，如果觉得还不错，还请在豆瓣上做个评论（https://book.douban.com/subject/30250772/，或点击“原文连接”）。

 