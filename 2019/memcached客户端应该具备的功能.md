memcached 只是一个服务，为了更好的使用它，必须从客户端的角度来审视它，很多客户端实现了很多 memcached 本身不具备的功能，优秀的 memcached 客户端应该具备哪些功能呢？

本篇文章从 PHP memcached 扩展的角度，看看它具备了哪些功能。

1：支持一致性 Hash 

一致性 hash 是一种更好分布key的模型，增加或删除某个实例的时候，不会导致过多的 key 失效。

常规hash算法，增加第11个实例，将有40%的key 失效，而使用一致性hash算法，只有10%的key 失效。

在 PHP memcached 扩展中，普通 Hash 算法由 Memcached::OPT_HASH 变量定义，分布式 Hash 算法由 Memcached::OPT_DISTRIBUTION 变量定义。

2：权重
 
不同 memcached 节点的能力不一定相同，所以在启用一致性hash的时候，需要给节点配置不同的权重。需要注意到是，各个客户端配置的节点顺序和对应权重必须一致，否则就会导致相同的key对应到不同的节点中。

3：序列化

memcached 服务不关心数据结构，它存储的就是字符串，如果用户想存储复杂的数据结构，客户端可以先序列化，然后 set，get 的时候再反序列化。

那么客户端在 get 的时候怎么知道是否要反序列化？客户端在 set 的时候需要对 item 设置特殊的 flag，这样 get 的时候才能正确反解。

PHP memcached 客户端支持三种序列算法，Memcached::SERIALIZER_PHP、Memcached::SERIALIZER_IGBINARY、Memcached::SERIALIZER_JSON。

4：压缩
 
为了有效利用内存和减少网络延迟，item 大小如果过大，客户端在 set 之前会压缩数据。

大部分客户端会根据阀值决定是否启用压缩，过小的item如果压缩只会消耗cpu，并不能减少内存的使用。

5：管理连接对象

如果频繁的打开和关闭memcached连接对象，可能会导致连接泄漏，所以大部分客户端都有管理memcached连接对象的方法。

比如 PHP memcached 客户端的 addServer() 函数不会直接连接 memcached 节点，只有具体 set/get 请求的时候才会连接。

6：Failure or Failover

在[《正确理解memcached，才能更好的使用》](https://mp.weixin.qq.com/s/njbvleUr8_PmhdEjZkpCIA)这篇文章中已经说过，尽量选择 Failure。

参考了 Laravel 代码，PHP 如果想支持 Failover，请使用下列代码：

```
$m = new Memcached;
$m->setOption(Memcached::OPT_DISTRIBUTION,Memcached::DISTRIBUTION_CONSISTENT);
$->setOption(Memcached::OPT_LIBKETAMA_COMPATIBLE,TRUE);
$m->setOption(Memcached::OPT_RETRY_TIMEOUT,2);
$m->setOption(Memcached::OPT_SERVER_FAILURE_LIMIT, 1);
$m->setOption(Memcached::OPT_AUTO_EJECT_HOSTS,true);
```

7：Multi-Get

客户端可以从多个 memcached 实例中获取多个 key 对应的数据，具体的实现方法取决于客户端的实现（比如客户端可以并行向不同的 memcached 节点发送请求）。

8：客户端应该能够存储二进制或字符串数据

PHP memcached 扩展默认没有开启二进制协议的支持，Memcached::OPT_BINARY_PROTOCOL 可以控制打开。
  
9：Noreply 

如果对 set 的结果不关心，为了提升性能，客户端不用等待 memcached 响应。

在 PHP memcached 扩展中，该功能默认没有开启，可以使用 Memcached::OPT_TCP_NODELAY 开启。

10：超时控制 

大部分 memcached 客户端支持连接超时（毫秒级别）、连接重试次数、发送超时、读取超时。

PHP memcached 扩展在这方面说的不是很详细，只有非阻塞模式，这些控制才能生效，参考下列代码：

```
$m = new Memcached;
//阻塞模式调用memcached
$m->setOption(Memcached::OPT_NO_BLOCK,FALSE);
$m->setOption(Memcached::OPT_CONNECT_TIMEOUT,2000);
$m->setOption(Memcached::OPT_POLL_TIMEOUT,2000);
$m->setOption(Memcached::OPT_RECV_TIMEOUT,750);
$m->setOption(Memcached::OPT_SEND_TIMEOUT,750);
```

11：支持从特定 memcached 节点获取数据

在启用一致性 hash 算法的时候，如果某个用户的数据有多个 key（姓名、地址、身高...），为避免分散存储，可以将关联的key存储到同一个节点上，PHP memcached 扩展的 setByKey() 函数支持该功能。

```
//指定某个key存储在特定的server上
$memcache->addByKey('server1','key','value1');
$memcache->addByKey('server2','key','value2');
$memcache->addByKey('server3','key','value3');
echo $memcache->getByKey('server1','key');
print_r($keys);
```

---

**memcached 系列文章：**

- [使用Memcached实现抽奖活动](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)  
- [简单理解memcached的内存分配](https://mp.weixin.qq.com/s/8fs5YU8drC5vUt1RxOgifw) 
- [聊聊memcached1.4的LRU算法](https://mp.weixin.qq.com/s/hfXWGm2fuyeThHawEHub-w)
- [memcached1.5更好的LRU算法，了解下maintainer线程](https://mp.weixin.qq.com/s/BG3wpLOWQJrKd0_btxo1Tw) 
- [memcached1.5更好的LRU算法，了解下crawler爬虫](https://mp.weixin.qq.com/s/p40CJOlTITU-__D4t05D7g)  
- [非memcached默认端口，wireshark如何解析它？](https://mp.weixin.qq.com/s/OxjqA3b8JDubZiHsFxtu-w)
- [正确理解memcached，才能更好的使用](https://mp.weixin.qq.com/s/njbvleUr8_PmhdEjZkpCIA)
- [从运维的角度理解memcached](https://mp.weixin.qq.com/s/ZyJjLMYmjNPeVq1HbU5wTQ) 

--- 

欢迎了解我的书[《深入浅出HTTPS：从原理到实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)，也可以关注公众号（ID：yudadanwx，虞大胆的叽叽喳喳）。