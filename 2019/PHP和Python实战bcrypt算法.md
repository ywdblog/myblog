本文是 bcrypt() 算法介绍的第二篇，第一篇参考[《安全存储口令的业界标准：bcrypt算法》](https://mp.weixin.qq.com/s/bBUuWwsg9qyNLo1MsbtTeQ)，本文主要介绍PHP和Python语言中如何更好使用 bcrypt 算法保护口令安全。

在一个系统中，可能有多种语言需要校验同一个口令密文，PHP和Python在操作上也是互通的。

在 PHP 语言中，已经不建议使用 bcrypt() 算法，推荐使用**Password Hashing Functions**。

这个库最重要的一个函数是 password_hash，原型：

```
string password_hash($password , int $algo [, array $options ])
```

$algo 表示使用那种 Hash 算法，默认是 PASSWORD_DEFAULT，表示 bcrypt 算法。

$options 可以有两个属性，cost 表示迭代因子次数，$salt 表示可以显示输入 salt，但从 php7 版本开始废弃 $salt 参数，也就是由函数生成不可猜测的 salt。

看看如何使用：

```
// PHP 5.5 以上版本
function better_crypt($input, $rounds = 7) {
    $options = [
        'cost' => $rounds,
    ];
    return password_hash($input, PASSWORD_BCRYPT, $options);
}

$hash = better_crypt("woshi123pas");
```
 
接下去**重点**看下口令密文的输出 $2y$07$xiJjq8T8RMl.4eIlQs3UhOGrm/WoO7.GcuQIZfOHCFdm0nVRP3WjC。

其中 $2y 表示采用 bcrypt() 算法，10 表示迭代因子，xiJjq8T8RMl.4eIlQs3UhOG 表示 salt，剩余部分才是口令密文。

等等，在[《如何安全存储口令？了解下Hash加盐的原理》](https://mp.weixin.qq.com/s/1X_aOoadk1CCBzGFC-EM3w)这篇文章中说过，salt 和口令密文应该分开存储，但很多文章在介绍 crpyt 算法的时候，都没有建议 salt 和 口令密文分开存储，从安全的角度，**请务必将迭代因子、salt、口令密文分开存储**。

如果想校验口令密文，那么可以使用 password_verify() 函数校验，贴代码：

```
 
// hash 是保存在数据库中的口令密文
function verify_crypt($input, $hash, $rounds = 7) {
    if (password_verify($input, $hash)) 
        return TRUE;
    return false ;
}

$hash = better_crypt("woshi123pas");
var_dump(verify_crypt("woshi123pas",$hash));
```

还可以更新口令密文，调用 password_needs_rehash() 即可，定期更新 salt，可以更安全保护口令。

最后介绍下 Python 中的 bcrypt 包，如果没有安装，使用 pip3 install bcrypt 安装。

贴代码：

```
import bcrypt

def bcryptfun(password):
        salt = bcrypt.gensalt(rounds=12)
        print(salt)
        hashed_passwd = bcrypt.hashpw(password, salt)
        return (hashed_passwd)
        
password = 'wo@yigepass'
password = bytes(password,'utf-8')
#加密口令
mw=bcryptfun(password)
print (mw)

#校验
mwjy = bcrypt.hashpw(password, mw) == mw
print(mwjy)
```

Python 和 PHP 校验口令密文是互通的，大家可以测试下。

口令保护系列文章：

- [如何安全存储口令？了解下Hash加盐的原理](https://mp.weixin.qq.com/s/1X_aOoadk1CCBzGFC-EM3w)
- [一个可实践的「Hash加盐」技术方案](https://mp.weixin.qq.com/s/WZQ4aJLnnSnKym5yBp3v2g)
- [安全存储口令的业界标准：bcrypt算法](https://mp.weixin.qq.com/s/bBUuWwsg9qyNLo1MsbtTeQ) 
- [PBKDF2函数，比「Hash加盐」更好的口令保护方案](https://mp.weixin.qq.com/s/16Rze8cPFfrHolBMRhznmA) 

--- 

了解我的书[《深入浅出HTTPS：从原理的实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)，如果觉得还不错，还请在豆瓣写个评论（https://book.douban.com/subject/30250772，或点击“原文连接”）。