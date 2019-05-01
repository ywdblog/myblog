基于memcached官方wiki，写了几篇 memcached 内部机制的文章，比如内存分配、LRU的工作原理。接下来从应用的角度说说memcached，只有从正确的角度理解，才能更好的应用，否则就会出现很多误解，比如有时候比较memcached和redis是毫无意义的。

1：内存、内存、内存

memcached是完全基于内存操作的，是一个缓存系统，从本质它不是一个数据库系统，也不支持持久化。

既然是内存，在使用之前问自己，memcached缓存的数据如果丢失，会影响你的业务吗？

2：分布式系统

memcached本身是一个非常轻量级的服务，不支持主辅同步，也没有集群的概念。但为了可扩展性，memcached 服务器端和 memcached 客户端结合起来可以构成一个分布式系统。

在memcached分布式系统中，各个 memcached 节点之间无须通信，所以扩展性非常好。

3：不支持复杂的操作

和 redis 不一样的是，memcached 没有复杂的数据结构（比如队列、集合），它只能存储字符串类型，也不关心具体存储什么，这是它的一个优势：简单。但很多人用的时候可能就不爽了，说它比 redis 弱爆了，其实双方的应用场景不一样。

当然也可以基于基本命令模拟一些数据结构，这是允许的，但这和memcached无关。

4：原子操作

memcached 所有操作都是 O(1) ，都是原子操作，同一时间操作多个 set 不会有任何的影响。

5：性能

memcached 性能高的原因主要在于 libevent 事件机制、多线程、全内存操作、模型简单（比如 O(1)，尽量避免锁机制）。

配置相对较高的服务器，每秒可以处理 200,000+ 的请求，即使在很慢的机器上，每秒处理几百次也毫无压力。

memcached 所有的操作都是 O(1)，set/get 操作不应该出现滞后，一个请求正常情况下不到 1ms 就能返回，如果遇到 Hang，那可能是连接数过多、产生了 swap、遇到了网络问题。
 
6：不推荐存储 session 数据

现在很多业务都用 memcached 存储 session 数据，在使用之前一定要评估风险，因为 memcached 是基于内存的，如果存储的 session 丢失了，会话用户就掉线了。

7：不支持命名空间

memcached 一般针对某个key进行操作，不支持 namespace 功能，原因在于为了实现namespace，会让系统实现复杂化（比如为了实现过期设置功能）。
 
但命名空间的作用却非常大，比如以下需求：

- 希望批量删除某些 key。
- 如果获取相关联数据（比如有多个key），但 memcached 节点非常多，如果这些key根据 Hash 算法分散存储到多个节点，那么获取的时候性能就会下降。 

但很多客户端模拟实现了namespace功能 ，比如 PHP memcached 扩展就有相应的函数，比如 addByKey()，getAllKeys() 等函数，这些后续我再说。

8：Failure or Failover

如果你的应用没有使用一致性hash算法或其他的hash算法，连接某一个memcache节点的时候，如果网络连接超时，或某个节点不能通信（比如网线被拔了），那么客户端调用只能选择 Failure，遇到这样的情况，可能就要去后端读取数据了。

如果你的应用使用一致性hash算法，那么应该选择 Failure or Failover？

这里有一个误区，一致性 hash 算法是减少上线或下线服务器带来的缓存失效问题，本质上不是为了让客户端随时随地都能获取到数据。

如果网络连接超时，或某个节点不能通信（比如网线被拔了），尽量**不要选择自动 Failover**，比如三个节点，本来某个 key 对应第一个节点，如果客户端发现第一个节点不可用，就自动存储到第二个节点，原因就在于如果第一个节点恢复了，可能获取到旧的数据（在 Failover 的时候新数据更新到第二个节点）。

根据这一原则，PHP memcached 扩展默认是不做 Failover 的，这也是正确的应用方式，如果你实在要 Failover，可以使用下面的代码。

```
$memcache = new Memcached;
$memcache->setOption(Memcached::OPT_DISTRIBUTION,Memcached::DISTRIBUTION_CONSISTENT);
$memcache->setOption(Memcached::OPT_SERVER_FAILURE_LIMIT, 1);
$memcache->setOption(Memcached::OPT_AUTO_EJECT_HOSTS,true);
$memcache->addServer('localhost', 11212);
$memcache->addServer('localhost', 11213);
$memcache->set("abc",1);
echo $memcache->get("abc");
print_r($memcache->getResultMessage());
```

9：能不能list所有的keys
 
不能，memcached 本身不支持，它就是一个 key-value 系统，但如果实在想实现list，可以通过两个办法：

- 推荐在启动的时候配置 -vvv，这样能够将所有set的key打印出来，而且是排查问题非常好的方法。
- 运行 stats cachedump 命令，不过不推荐使用，因为操作比较慢，而且只能拿到局部的数据。
 
10：memcached 应用场景？

- 能缓存任何数据，有效减少后端的压力并提升响应能力。
- 计算器，比如利用 incr 命令。
- Rate limiting，非常有用，比如登陆系统可以使用它限制非法登陆。
- 虽然 memcached 不支持队列和堆栈，但 append/prepend 命令还是有一定作用的，比如可以实现排行榜。

11：一些小技巧

（1）Zero byte values
 
如果你只是要使用 memcached 的key ，其对应的value没有任何价值，那么为了减少内存的使用，可以设置0字节长的值。

（2）尽量减少 key 长度，有效利用内存。
 
（3）有效利用锁机制，比如 add 命令，专门写过一篇文章介绍过[《使用Memcached实现抽奖活动》](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)。

**memcached 相关文章：**

- [使用Memcached实现抽奖活动](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)  
- [简单理解memcached的内存分配](https://mp.weixin.qq.com/s/8fs5YU8drC5vUt1RxOgifw) 
- [聊聊memcached1.4的LRU算法](https://mp.weixin.qq.com/s/hfXWGm2fuyeThHawEHub-w)
- [memcached1.5更好的LRU算法，了解下maintainer线程](https://mp.weixin.qq.com/s/BG3wpLOWQJrKd0_btxo1Tw) 
- [memcached1.5更好的LRU算法，了解下crawler爬虫](https://mp.weixin.qq.com/s/p40CJOlTITU-__D4t05D7g)  
- [非memcached默认端口，wireshark如何解析它？](https://mp.weixin.qq.com/s/OxjqA3b8JDubZiHsFxtu-w)

--- 

 我的书[《深入浅出HTTPS：从原理到实战》]((https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)，如果觉得写的还可以，欢迎在豆瓣做个评论（https://book.douban.com/subject/30250772，或点击“原文连接”）；也可以关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳）。