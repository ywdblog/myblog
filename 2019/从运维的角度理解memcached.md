[《使用Memcached实现抽奖活动》](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)从应用的角度讲了讲memcached，这篇文章从运维的角色说一说，换个角度思考能够更好的使用memcached。

1：启动

memcached 在启动的时候有很多的参数，列举几个重要的。

-m：表示 memcached 可以使用的内存，注意 memcached 实际占用的内存大于 -m 配置的值，比如网络连接也要占用内存。

-d：表示在后台运行memcached。

-v：控制 STDOUT/STDERR 的输出，多个 -v 可以输出很多有用的数据，比如查看set了那些key，对于排查问题非常有用。

-l：表示服务绑定的网卡地址，绑定内网网卡可以有效保证安全性。

-p：修改 tcp 监听端口，对应的 udp 端口也修改。

-U：是否支持 udp 端口，默认是关闭的。

-u：以某个用户的身份启动 memcached，避免使用 root 启动，有效保障安全性。

-s：严格限制只有本地 user 通过 unix domain socket 访问 memcached。

-c：设置最大连接数，默认是 1024，如果当前最大连接数超过 -c 设置的值，请求在连接队列中等待，由于 memcached 连接占用的资源非常少，所以该值不要设置太高。

-t：工作线程数量，默认是 4，可以有效利用多核服务器，由于 memcached 性能很高，该值不建议设置过高。

2：硬件选择

从硬件的角度思考 memcached，以便选择正确的硬件，对某个系统参数进行调优。

（1）cpu or 内存 

memcached 是多线程模型，性能极高，所以不一定要多核服务器，但内存越大越好。

那么选择什么样的服务器来放置 memcached 呢？如果规模较小，那么在 web 服务器上部署 memcached 是非常不错的选择，memcached cpu 消耗很小 ，不会加重 web 服务器的负载，而 web 服务器对内存的要求不高，剩下来的内存可以供 memcached 使用。

需要注意的是，不要在专用的数据库服务器上部署 memcached，因为这两个服务对内存的要求都非常高，避免发生 swap。

对于具有一定规模的业务来说，采用专门的服务器来部署 memcached 更有优势，它们都具有大容量的内存，需要注意的是，一个节点可以部署很多个实例，如果某个节点挂掉了，那么失效的 key 会非常多，这在容量规划的时候要注意。

（2）网卡

网络流量取决于memcached请求峰值和item平均大小，避免跑满带宽，不过现在的网卡都是千兆了，理论上应该不会出现问题。

（3）网络连接数
 
首先要注意的是，-c 启动的连接数不要太高，一方面避免超过系统允许的连接数；另外一方面memcached非常高效，没有必要设置太高，-c 最优化配置和操作系统本身的TCP优化很有关系。

memcached使用专门的一个线程接收客户端连接，如果客户端频繁的连接、释放，这个线程的压力会非常大，可以使用长连接或memcached udp 监听来避免连接和释放的消耗。
 
3：分析和监控

memcached 协议命令有两种类型，add/set/get主要用于完成工作，而其他一些内部命令对于排查、优化、分析都非常有帮助，比如：

- stats：**常规统计**，列出的信息非常多。
- stats slabs：**slab 统计**，返回大小和内存使用情况，也包含每个 slab class的统计信息。
- stats item：**item 统计**，返回每个slab class 的item信息，相比 stats slabs 命令，stats items 显示更高层次的信息。 
- stats sizes：**item 大小统计**，返回不同大小item存储的个数。

了解这四个基本命令后，我们看看监控怎么弄，当然也可以看看官方提供的一个工具（https://github.com/memcached/memcached/blob/master/scripts/memcached-tool），并不是为了使用它，而是看看它关注什么，从中可以学到很多。

**提醒：**memcached 1.4版本以后发生了很大的变化，内部命令和内存管理发生了很大的变化，大家如果使用了一些外部统计和监控工具，需要看看这些工具有没有更新。

（1）应用监控，定时执行 set、get 等命令，监控操作是否成功，以及记录响应的性能。如果连接很慢，可能是网络问题，如果性能低下，可能产生了内存 swap。

（2）stats 

curr_connections：查看某个端口的连接数，避免 curr_connections 大于配置的最大连接数（-c）。

listen_disabled_num：这个值最好接近于 0，如果大于0，表示有多个连接进入了连接队列，需要等待其他连接的释放。

accepting_conns：这个数值和 listen_disabled_num 很类似，如果服务连接数已达到最大值，该值被设置为 1。

limit_maxbytes：该值等于服务启动配置的内存总容量（-m），即有多少内存可以使用。

cmd_flush：该数值表示有多少个 flush 命令被执行了，如果数值过大，说明应用的模式有问题，从监控的角度看，应该尽量避免。

（3）查看 evicted 

可以使用 stats 和 stats items 命令监控 evicted 和 evicted_nonzero 数量，前者针对整体，后者针对各个 slab。

evicted 表示被剔除的数量，evicted_nonzero 更重要，值如果过高表示没有过期的 item 被 evicted 的太多了，这可能会影响业务。

**注意：**evicted 过高，不一定代表内存不够。比如 set 了大量 keys，但这些 keys 后来从没使用过，也会导致 evicted 过高。

（4）查看命中率 
 
通过 stats 命令可以统计全局的命中率：get_hits / (get_hits + get_misses，了解某个时间点命中率，以及命中率变化的情况。

stats slabs 命令能够了解每个 slab 的统计情况，但没有 get_misses 数值，但可以通过观察 get_hits 和 cmd_set 间接了解每个 slab 命中率的变化。
 
**注意：**命中率不高，不一定代表内存不够，比如请求的 keys 大量不存在，也会导致命中率过低。

（5）系统监控

比如系统负载，cpu 使用率、swap 情况、网卡流量等等。

**memcached 相关文章：**

- [使用Memcached实现抽奖活动](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)  
- [简单理解memcached的内存分配](https://mp.weixin.qq.com/s/8fs5YU8drC5vUt1RxOgifw) 
- [聊聊memcached1.4的LRU算法](https://mp.weixin.qq.com/s/hfXWGm2fuyeThHawEHub-w)
- [memcached1.5更好的LRU算法，了解下maintainer线程](https://mp.weixin.qq.com/s/BG3wpLOWQJrKd0_btxo1Tw) 
- [memcached1.5更好的LRU算法，了解下crawler爬虫](https://mp.weixin.qq.com/s/p40CJOlTITU-__D4t05D7g)  
- [非memcached默认端口，wireshark如何解析它？](https://mp.weixin.qq.com/s/OxjqA3b8JDubZiHsFxtu-w)
- [正确理解memcached，才能更好的使用](https://mp.weixin.qq.com/s/njbvleUr8_PmhdEjZkpCIA)
- [从运维的角度理解memcached](https://mp.weixin.qq.com/s/ZyJjLMYmjNPeVq1HbU5wTQ) 

--- 

欢迎了解我的书[《深入浅出HTTPS：从原理到实战》，也可以关注公众号（ID：yudadanwx，虞大胆的叽叽喳喳）。