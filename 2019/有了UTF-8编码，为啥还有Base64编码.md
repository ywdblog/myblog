你是不是遇到过以下的一些情况，比如 Linux 终端打开一张图片的时候会出现“乱码”（将终端控制台搞乱）；运行 `openssl rand 100` 的时候也会出现“乱码”；在 HTML 页面内嵌图片的时候为什么要进行 Base64 编码？今天我想把这个问题说清楚。

在解释之前，先说说文本文件和二进制文件的区别。对于文本文件，在编辑器输入的字符其实都是一个个 Unicode code points，为了存储这些字节序列，保存的时候必须对字符进行编码，比如 UTF-8 编码。或者说文本文件就是 Unicode code points 序列组合。

对于文本文件，大部分软件（比如 cat，grep）都能直接解析和呈现，不会出现“乱码”问题。

二进制文件也是字节组合，解析二进制文件必须选择正确的解释器，比如 word 能够解析 .doc 后缀文件，如果你直接用 cat 打开，那么就会出现“乱码”，因为 cat 不知道二进制文件的编码规则。

HTML 网页可以认为是文本文件，而图片就是二进制文件。如果用 cat 直接打开二进制文件（很少有人这么做），会对内容进行 Base64 编码，虽然操作者还是不明白这些字符代表的含义，但至少不会搞乱控制台了。

用一句话说，Base64 编码是用来编码二进制文件的，这样在呈现和传输的时候不会遇到问题。

那 Base64 编码来源于哪儿呢？Base64 编码是 MIME 扩展的一部分，在 Web 和 Mail 的早期，传输的都是文本，而且都采用 ASCII 字符，根本不会遇到编码问题。

后来邮件（Web 也是）可以包含非 ASCII 字符了，就出现了 Content-Type MIME 头，比如 `Content-Type：text/html;charset=utf-8` 表示这个类型文件的编码是 utf-8， Content-Type 更多说明这个文档的类型。

那一个文件在邮件系统或 Web 中传输的时候，采用什么编码呢？默认就是 7 bit，它是 Content-Transfer-Encoding MIME 头的一个属性值。

什么意思呢？只要是文本数据，不管其选择什么编码，传输编码（Content-Transfer-Encoding）都可以采用 7bit，7bit 其实相当于没有编码。7 bit 编码是邮件标准和 Web 标准中唯一一个安全传输的编码。想想看，一个 UTF-8编码的文档在 Web 传输的时候是拆分成一个个 7bit 传输的（可以这么理解）。

很多同学说，编写 Web 代码的时候，我根本没有看到 Content-Transfer-Encodin 头，其实默认就是 7bit，是为了向下兼容。

既然 7bit 相当于没有编码，那要它何用？Content-Transfer-Encoding 还有个属性值就是 Base64，如果你想在一个邮件（Web 也是）中附带一个文件（二进制文件），那么这个文件必须进行 Base64 编码。

有的同学说，你刚才不是说 7 bit 编码是邮件标准和 Web 标准中唯一一个安全传输的编码，那 Base64 编码怎么也能安全传输？因为 Base64 编码可以将数据转为 7 位的形式，这样就相当于一个 7bit 编码了，从而符合网络传输标准了。

所以，在编写一个 HTML 网页的时候，就可以对二进制图片进行 Base64 编码然后放进 HTML 页面中，这样就符合网络传输标准了。如果你直接嵌入二进制文件，那么就无法传输和解析了。

Base64 编码并不是为了让你肉眼看的，而是为了传输使用，还能将其转化为 7 bit 字符，这样就不会弄乱控制台了。

Base64 能对任何文件进行编码，缺点就是编码后占用的空间比原始文件大了1/3。

其实 Email 历史非常有意思，MIME 是 Web 的鼻祖，包括 Content-type、Content-Transfer-Encoding、Content-Disposition 等头非常重要。

如果让你拼装一个混合类型的邮件，你会弄吗？首先要指定全局 Content-type 类型为 multipart/mixed；而如果附带二进制文件，则必须使用 Content-Transfer-Encoding 头的 Base64 进行编码，以便让它符合互联网传输标准；Content-Disposition 决定了附件如何呈现出来（下载还是内联显示）。
 
关联文章：

- [浏览器下载中文名文件出现乱码，如何解决？](https://mp.weixin.qq.com/s/dADwiThHFf87SppGVeLDXA)
- [你真的知道网页上传文件背后的原理吗？](https://mp.weixin.qq.com/s/C3H6eNmfBmdX7urHS5dLwA) 

--- 

欢迎关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），一直在用心写。