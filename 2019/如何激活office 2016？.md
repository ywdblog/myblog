今天排查问题的时候，要编辑一个 excel 文件，发现自己的 office 2016 已经到期了，没办法激活了，简单看了几篇文章，研究了一些解决方案（属于破解，并不提倡），另外网上的一些文章也有很多的错误，本文尽量以自己的简单理解把它说清楚。

主要的解决方案就是利用微软的 KMS（Key Management System）激活漏洞，KMS 主要用来激活 VOL 版本的 Windows 和 Office。

什么是 KMS 呢？就是微软为一些企业版操作系统和软件采取的一种授权方式，无须用户输入 license key。

那么那些软件版本能够采用 KMS 激活呢？批量许可版本（Volume licensed versions，VOL）都可以，VOL 内置通用批量许可密钥 (GVLK)，拥有 GVLK 的软件就能采用 KMS 激活。

从理解上看，只有企业版才是 VOL 版本，实际上并不是这样（可能理解的不透彻），不然也就无法激活我的 office 2016 专业版了。

对于 office 来说，有很多类型的版本，比如说学生版，企业版，专业版，Office 365，其中 Office 365 在我的印象中，是需要使用微软帐号登录的，相当于一个云端的 office 产品。

从另外一个角度来看，office 还分为 VOL 版本和个人版，按道理我使用的是个人版（专业增强版，cn_office_professional_plus_2016_x86_x64_dvd），无法通过 KMS 激活，但专业版确实存在 GVLK（属于 VOL 专业版？），仍然可以激活，可以通过[GVLKs for KMS and Active Directory-based activation of Office 2019 and Office 2016](https://docs.microsoft.com/en-us/DeployOffice/vlactivation/gvlks) 查看你的 office 是否支持激活。

如果想安装 office 2016，可以使用百度网盘打开下面两个链接中的一个：

- VOL 版：ed2k://|file|SW_DVD5_Office_Professional_Plus_2016_64Bit_ChnSimp_MLF_X20-42426.ISO|1123452928|31087A00FF67D4F5B4CBF4AA07C3433B|/
- 专业版：ed2k://|file|cn_office_professional_plus_2016_x86_x64_dvd_6969182.iso|2588266496|27EEA4FE4BB13CD0ECCDFC24167F9E01|/
 
确定你的 office 能够通过 KMS 激活，接下去寻找能够破解 KMS 的软件了，比较流行的就是 KMSpico，而且这些软件做的越来越流弊了，只要简单的安装，就能够激活 office 了。

看看它的特性：

- 100% clean tool：官方介绍它是一个非常干净的工具，不会危害你的电脑，所以建议安装英文版的 KMSpico。
- permanently activate any version of Windows or Microsoft office：能激活任何版本的 office，这和上面介绍的认知（只能 VOL 版本激活）有很大的不同，不知道具体的原理，只能这样猜测：有 GVLK 的 office 和操作系统都能激活。
- emulated instance of a KMS server on your machine：什么意思呢，很多 KMS 破解工具会让你配置一个 KMS 网络服务用于激活，而 KMSpico 会在你的本机运行一个 KMS 服务器，没有网络也能激活，也证明它的安全性比较高。
- only lasts for 180 days：180 天激活会失效，但 KMSpico 每两天会激活一次，避免 180 天后失效。（遗留一个问题，我激活后把 KMSpico 删除是否可以?）

KMSpico 属于“非法软件”，杀毒软件和windows defender 安全中心会认为该软件是病毒软件，在安装和激活的时候必须关闭它们，由于我没有安装杀毒软件（总觉得他们就是病毒软件），所以主要做的就是关闭 windows defender 安全中心，如下图：

![](4-kms-aqzx.png) 

我尝试为 KMSpico 开一个白名单（对它不采取任何防护措施），但没有成功，配置如下图：

![](4-kms-pcx.png) 

接下来就是从 KMSpico 官网下载（不要用chrome下载，会阻止，使用IE，很讽刺把），然后运行即可（包含 KMSpico 和 Autopico 程序），如果没有运行 Autopico 程序，他们不会在后台运行，相对是安全的，但也无法做到180天后自动激活的，为了进一步安全，激活 office 后就卸载了 KMSpico，反正 180 天后再激活一次就可以了。

最后建议不要下载国内版本的 KMSpico，不知道会不会有后门。

---

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），相信不会失望的。