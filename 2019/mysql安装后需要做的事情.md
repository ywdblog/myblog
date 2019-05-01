在[《在ubuntu中安装mysql的一些记录》](https://mp.weixin.qq.com/s/GHz8EG_jC4ABgBx6j_aaVw) 这篇文章介绍了如何安装mysql，接下去思考一个问题，安装后我应该做些什么呢？如何确保mysql服务是安全的呢？这篇文章借鉴官方文档理了下mysql安装后需要做的事情，总共有四个步骤。

**第一个步骤**就是初始化目录，在Debian（包括Ubuntu）安装过程中，这个步骤是自动完成的，但我觉得初始化步骤对于理解mysql其实挺重要的。如果你使用源码安装，或者觉得自己的mysql环境被破坏了（非线上业务），都可以通过该步骤恢复到初始状态。

初始化目录主要是了初始化系统数据库（数据库名为 mysql）和进行一些授权。

首先找到mysql的datadir目录，至于如何找到，可以进入mysql命令行执行 `select @@datadir`，在ubuntu上地址默认是 /var/lib/mysql。

然后删除这个目录下的所有文件，并停止 mysqld 服务。确保 /var/lib/mysql 目录的属组和属主是 mysql（不要使用 root 权限）。

接下去执行下列命令：

```
$ mysqld --initialize-insecure --user=mysql
```

特别说明，mysql早期版本的datadir目录初始化脚本是 mysql_install_db，现在统一使用 mysqld，mysqld 也是mysql内置的服务。

其中 --user 参数表示以某个 linux 系统账户初始化目录，如果想变更 datadir 目录，可以显示的加上 --basedir 和 --datadir 参数。

--initialize-insecure 也可以使用 --initialize 参数代替，前者生成的 root 口令为空，而后者生成一个随机的口令（通过命令行输出能够看到）。 

**第二个步骤**就是启动mysql服务，确保安装没有问题，在 Debain 系统下，一般执行下列命令：

```
$ mysqld_safe --user=mysql & 
```

如果使用 RPM 包安装的，执行下列命令：

```
$ systemctl start mysqld 
```

至于 mysqld_safe 和 mysqld 的区别，后续再说，可以简单认为 mysqld_safe 是 mysql 的一个外壳程序。

**第三个步骤**是测试mysql服务是否正常，其实很简单，就是看看mysql状态、版本等信息，具体就是mysql内置的一些程序，比如：

```
$ mysqladmin version 
$ mysqladmin shutdown 
$ mysqladmin variables
$ mysqlshow 
$ mysql -e "SELECT User, Host, plugin FROM mysql.user" mysql 
```

在执行的时候，都需要root密码（当然root密码可以为空，具体见第一个步骤）。

**第四个步骤**很关键，就是初始化你的root密码，避免带来安全问题。在安装mysql的时候，root密码是如何创建的？下面简单列举下：

- 如果在windows中安装，安装程序会有选项让你配置密码。
- 如果使用RPM包安装，会生成一个随机密码，密码值在服务器error log 中。
- 如果在Debian中安装，会通过选项让你自己选择一个密码，但我在Ubuntu 16 中并没有，具体见[《在ubuntu中安装mysql的一些记录》](https://mp.weixin.qq.com/s/GHz8EG_jC4ABgBx6j_aaVw) 这篇文章。
- 手动初始化目录的时候，也有针对密码的处理，具体见本文第一个步骤中的描述。

root 密码（在 mysql.user 表）非常重要，它拥有所有的权限，所以应该谨慎。

不管密码是什么，建议第一次启动后，重新初始化，执行下列命令即可：

```
$ ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
```

启动过程中可能会遇到很多问题，这个有机会再讲，因为有很多基础知识还没说，如果提前解释启动过程中的一些问题，反而会影响理解。

**相关文章：**

- [MySql显示乱码的解决方法](https://mp.weixin.qq.com/s/tMMwwFts2_cT2bnvJGKZEA)  
- [在ubuntu中安装mysql的一些记录](https://mp.weixin.qq.com/s/GHz8EG_jC4ABgBx6j_aaVw)

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），一直在用心写。