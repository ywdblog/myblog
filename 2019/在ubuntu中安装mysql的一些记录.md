前几天要测试个东西，需要用到 mysql，突然发现自己的机器上居然没有安装 mysql 服务，但对于 web 开发来说，其实掌握 mysql 还是很有必要的，现在分工越来越细了，云端的产品也越来越多了，开发人员有的时候处于温室当中，可能装个 mysql 也很费劲了。

为了温习 mysql，自己根据官方文档，在自己的阿里云 ECS（Ubuntu 16）中装了个 mysql 5.7 主流版本，用的是包安装方式，即官方的 MySQL APT，如果你也是初次学习 mysql，那么这篇文章很适合你。

安装分为多种情况：

（1）全新安装，比如机器上从来没有安装过mysql，没有历史包袱，安装简单轻松。

（2）机器原来采用源码安装过mysql，现在要使用 MySQL APT 安装，这种情况没有测试过，但类似于全新安装。

（3）机器原来使用 MySQL APT 安装过旧版本，现在想升级，这种情况官方也说的比较清楚。

（4）机器原来采用 MYSQL 非官方 APT 安装过，现在想使用 MySQL APT 替换，这种情况基本也能做到无缝升级。

（5）机器原来使用 APT 安装过 MYSQL 其他的分支（比如 MariaDB），如果想使用 MYSQL APT 升级，无法做到透明升级，需要采取另外的升级策略，尤其要先备份数据。 

> The MySQL APT repository can only replace distributions of MySQL maintained and distributed by Debian or Ubuntu. It cannot replace any MySQL forks found either inside or outside of the distributions' native repositories. 

本文主要以全新安装的方式介绍，也就是上述（1）和（2）两种情况。

如果你想全新安装，而且只是测试，那么先卸载 mysql，能避免很多问题：

```
#查看本机安装了哪些 mysql 组件
$ dpkg -l | grep mysql | grep ii 

$ apt-get remove mysql-server 
$ apt-get autoremove 

#卸载对应的 mysql 多个组件
$ apt-get remove package-name
```

首先下载一个 DEB 包（地址：https://dev.mysql.com/downloads/repo/apt/），这个包会根据操作系统自动化配置本地的 apt mysql 仓库。

这个包可以在下列平台使用：

- Debian 8 and 9
- Ubuntu 16.04, 18.04, and 18.10

然后运行下列命令配置：

```
$ dpkg -i mysql-apt-config_0.8.12-1_all.deb 
$ apt-get update
```
 
在安装的时候，会让你选择采用哪个版本的 mysql，以便配置 apt mysql 仓库，我选择的是 mysql 5.7，安装完成后，在 /etc/apt/sources.list.d 目录下出现 mysql.list 文件，内容如下：

```
deb http://repo.mysql.com/apt/ubuntu/ trusty mysql-apt-config
deb http://repo.mysql.com/apt/ubuntu/ trusty mysql-5.7
deb http://repo.mysql.com/apt/ubuntu/ trusty mysql-tools
#deb http://repo.mysql.com/apt/ubuntu/ trusty mysql-tools-preview
deb-src http://repo.mysql.com/apt/ubuntu/ trusty mysql-5.7 
```

可以手动编辑 mysql.list，如果你想采用 mysql 5.6 版本，可以运行下列命令，重新选择：

```
$ dpkg-reconfigure mysql-apt-config
```

一旦启用了 MYSQL APT，非官方的 mysql apt 将无法使用，这一点是要注意的。

接下去就是安装 mysql 了，执行下列命令：

```
$ apt-get install mysql-server 

# 查看 mysql 是否运行。
$ service mysql status
$ service mysql start
```

根据官方文档的描述，在安装 mysql 的时候会让你输入 root 的登录密码，但在我的 Ubuntu 16 上却没有交互式的提示让输入密码，找了下解决方案，下面简单描述下，感觉并不是最好的解决方案。

打开 /etc/mysql/debian.cnf 文件：

```
[client]
host     = localhost
user     = debian-sys-maint
password = password
socket   = /var/run/mysqld/mysqld.sock 
```

用debian-sys-maint账户可以成功登录 mysql，而且权限是极高的，如果你想让你的服务更安全，运行：

```
$ mysql_secure_installation 
```

脚本会交互式的问你是否重置密码、移除匿名用户、移除 test 数据库。运行该脚本，需要 root 用户的权限（开始也可以使用 debian-sys-maint 账户）。

记得以前安装 mysql 的时候，在 CentOS 系统下，需要手动运行 `mysql_install_db` 或 `mysqld --initialize` 脚本用于初始化目录，在 Debain 系统下（包括 Ubuntu），这一步在安装 mysql 的时候会自动完成。

需要注意的是，在不同的平台（比如 CentOS），安装方式是不一样的，不能照搬硬套，而且网上很多文章由于年代的原因，可能并不合适了，建议参考最新的官方文档。

- [MySql显示乱码的解决方法](https://mp.weixin.qq.com/s/tMMwwFts2_cT2bnvJGKZEA)  

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），一直在用心写。
