同学们，如果你看过我的书[《深入浅出HTTPS：从原理的实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA) ，而且还觉得不错，希望能在豆瓣上做个评论（https://book.douban.com/subject/30250772，或点击“原文连接”）

--- 

上个星期，一个对外的项目遇到浏览器下载附件错误的问题，简单的研究了下，涉及了二个问题：一个是通过编程如何让浏览器支持文件下载，第二个问题是如果附件是中文名称，如何避免文件出现乱码，这篇文章主要讲解文件下载的原理。

从本质上讲，为了支持文件下载，客户端（浏览器）和服务器支持**特定的HTTP头**即可，但不是所有的客户端（浏览器）都按照**标准**处理，比如说iPhone就无法通过浏览器下载文件。

在 **rfc7231** 中（HTTP/1.1 子协议 Semantics and Content）中已经说明 Content-Disposition 头并不是 HTTP 协议中的标准头，但 Content-Disposition 头在 HTTP 应用中有一定的应用场景，所以 **rfc6266** 详细描述了这个头部在 HTTP 中的使用标准，确切的说在 HTTP 应用中，这个头部称为 response header。

那么是否意味着 Content-Disposition 这个头部还有其他用途？是的，最早这个头在邮件应用中使用的比较多，属于 MIME 的一部分，Content-Disposition header（不是 response header）定义在 rfc2183 上，我在写[《你真的知道网页上传文件背后的原理吗？》](https://mp.weixin.qq.com/s/C3H6eNmfBmdX7urHS5dLwA) 文章时，对于 MIME 中如何使用 Content-Disposition 有所涉及。

我参考 CodeIgniter PHP 框架，研究了文件下载的原理，实际上就是输出一些 HTTP 头部，如下图：

- [25-httpddownload-postman.png](25-httpddownload-postman.png) 

大部分头部在常规的 HTTP 应用中（比如页面和接口）很常见，比如 Content-Length 表示附件的大小（在PHP中使用 strlen()算对吗？）；Cache-Control和Expires控制缓存头，对于下载附件来说，就是强制获取最新内容；Content-Type表示附件的类型，不过如果你统一使用 application/octet-stream 也没有问题，浏览器照样能够下载；application/octet-stream 头部代表未知的应用（二进制文件）。

Content-Transfer-Encoding 这个头部是第一次看见，它和 Content-Type 头部有点类似，但主要用于传输。它在 multipart MIME 类型中用的比较多（如果一个实体是 multipart 类型，那么 Content-Transfer-Encoding 的值只能是 "7bit"、"8bit"、"binary"中的一个），对应到文件下载，这个值设置为 binary，表示允许非ASCII码进行传输。可以认为 application/octet-stream 等同于 Content-Transfer-Encoding:binary 。关于这个头部，理解的不是很好，可以参考 **rfc2045**（内容实在太多了）。

对于文件下载，最重要的HTTP头部就是 Content-Disposition，它用来表示内容如何在浏览器中显示：可以是内联（inline）呈现（对应的内容将作为页面的一部分呈现）或者附件下载（attachment）的形式。其中 filename 参数表示附件的名称，如果是中文，在不同的浏览器可能会出现乱码（这个我下一篇会简单说一说）。

本质上附件下载就是这么简单，但在 RFC 中没有清晰的看到实现附件下载的说明。

 