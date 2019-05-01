最近在写 memcached 系列文章的时候，为了了解客户端和memcached之间的通信，使用wireshark分析了相关流量，但遇到了一个问题，觉得挺典型的，为此分享一下。

memcached 为了分析 memcached 流量，需要特定的解码器（默认是支持的），可以通过下列方法开启或关闭，点击【分析】菜单，选择【启用的协议】，如下图：

![memcached-qy.png](memcached-qy.png)

接下去说具体的问题：

memcached 默认端口是 11211，wireshark 在分析 memcached tcp 流量的时候，如果发现端口是 11211，则会解析出 memcached 协议数据，如下图：
 
![memcached-11211.png](memcached-11211.png)

如果 memcached 启动的端口非 11211，则 wireshark 默认不会显示 memcached 协议，只会显示 TCP 协议，如下图：

![memcached-50028.png](memcached-50028.png)

这样在排查问题的时候不是很方便，如何解决呢？有两个办法，接下去使用 wiershark 2.6.0 中文版演示。

1：tcp 流

右键点击 wireshark 界面，选择追踪流->TCP 流，就可以看到完整的 memcached 操作命令了。

如下图：

![memcached-50028.png](memcached-tcp.png)
 
2：变更解码方式

然后点击【分析】菜单，选择【解码为】，如果没有特定 memcached 端口的解码方式，可以手动添加（在本例中 memcached 启动端口是 50028），如下图：

![memcached-jiema.png](memcached-jiema.png)

然后你就能看到非 memcached 默认端口的解析了，如下图：

![memcached-50028-ok.png](memcached-50028-ok.png)

触类旁通，分析其它协议的时候，可以手动变更解码方式。

--- 

欢迎大家关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳）。