最近写了一些 Python3 程序，还是习惯在 Linux 下用 Vim 编码，可自己 Vim 也没有用什么插件，总感觉效率不好，就想着捣鼓下 Vim，弄个 Python IDE。

遇到的第一个问题让 Vim 支持 Python3，这样就能支持一些 Python3 程序或软件，看上去挺简单，没想到最后花了长时间，简单和大家分享下。

心得就是越折腾，理解的就越透彻，使用 Linux 解决问题不能死板硬套，一定要注意特定版本的 Linux 发行版，特定的环境，特定的软件版本。

我使用的操作系统是 Ubuntu 16。默认安装的 Vim 是 7.4 版本，本次打算升级到 8.1 版本。如何知道 Vim 默认支持的 Python 版本？输入下列命令就能知道：

```
$ vim --version

-L/usr/lib/python2.7/config-x86_64-linux-gnu -lpython2.7
```

初步的打算就是安装一个新的 Vim（不是替代 Vim 旧版本），新版本支持 Python3 即可。如何让 Python2 和 Python3 共存，可以见[《是时候配置一个Python3的开发环境了》](https://mp.weixin.qq.com/s/0gEB5g_OkFt58VroW96mbw) 

首先安装各类依赖包，不同的发行版各不相同，随便 Google 就能找到：

```
$ apt-get install  
    libcairo2-dev libx11-dev libxpm-dev libxt-dev  \
    python3-dev ruby-dev  libperl-dev git
```

接下去很多文章告诉你这样配置：

```
$ git clone https://github.com/vim/vim.git
$ cd vim
$ ./configure --with-features=huge \
    --enable-multibyte \
    --enable-rubyinterp=yes \
    --enable-pythoninterp=yes \
    --with-python-config-dir=/usr/lib/python2.7/config \
    --enable-python3interp=yes \
    --with-python3-config-dir=/usr/lib/python3.5/config \
    --enable-perlinterp=yes \
    --prefix=/opt/vim8
```

首先需要注意的是 Vim 不能同时支持 Python2 和 Python3，所以 --enable-pythoninterp=yes 参数要去除。

--enable-python3interp 参数表示启用 Python3 解析器。

最重要的就是 --with-python3-config-dir 参数，那么 Python3 的配置文件在哪儿呢？可以输入下列命令找到：

```
$ /usr/local/bin/python3-config --configdir

/usr/local/lib/python3.7/config-3.7m-x86_64-linux-gnu
```

其实高版本的 Vim 比较智能，根本无需 --with-python3-config-dir 参数：
 
```
$ ./configure --help | grep python

  --enable-pythoninterp=OPTS   Include Python interpreter. default=no OPTS=no/yes/dynamic
  --enable-python3interp=OPTS  Include Python3 interpreter. default=no OPTS=no/yes/dynamic
  --with-python-command=NAME  name of the Python 2 command (default: python2 or python)
  --with-python-config-dir=PATH  Python's config directory (deprecated)
  --with-python3-command=NAME  name of the Python 3 command (default: python3 or python)
  --with-python3-config-dir=PATH  Python's config directory (deprecated)
```

可以看出 --with-python3-config-dir 参数已经废弃了，所以只要 --enable-python3interp 参数就可以，安装脚本能够自行找到 --with-python-config-dir。

最后，我使用下令脚本完成安装：

```
$ git clone https://github.com/vim/vim.git
$ cd vim
$ ./configure --with-features=huge \
    --enable-multibyte \
    --enable-rubyinterp=yes \
    --enable-python3interp=yes \
    --enable-perlinterp=yes \
    --prefix=/opt/vim8
```

执行：

```
$ ln -s /opt/vim8/bin/vim /sbin/vim8 
``` 

这样输入 vim8 命令，就能打开 Vim 8.1。

在本次折腾过程中，我开始没有找到 Python3.7 python3-config，所以用的是 Python3.4，可始终记不起来，自己安装过 Python3.4，实际上 Python3.4 是 Ubuntu 默认的最小化安装，输入下列命令就会知道：

```
$ dpkg-query -L python3.4 

python3-minimal/trusty,trusty,now 3.4.0-0ubuntu2 amd64 [installed]
python3.4-minimal/now 3.4.3-1ubuntu1~14.04.5 amd64 [installed,upgradable to: 3.4.3-1ubuntu1~14.04.7]
```

从中可以看出，我从 ubuntu 14 升级到 ubuntu 16，默认使用 python3.4-minimal 包安装的，为了让操作系统保持干净，我将 Python3.4 删除了：

```
$ apt-get purge --auto-remove python3.4
```

另外一个问题是出现 PyThread_start_new_thread 报错（配置 python3.4），即使执行 `make clean` 重新安装也不行，最后全部删除再 git clone 后解决（配置 python3.7）。

安装完成后，打开文件发现二个问题，一个就是 Vim 配色全没了，另外一个就是 Backspace 键失效，最后编辑 ~/.vimrc 解决：

```
syntax on
set backspace=indent,eol,start
```

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），一直在用心写。