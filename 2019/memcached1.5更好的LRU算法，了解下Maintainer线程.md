从 Memcached1.5 开始，实现了一个改良的 LRU 算法，也叫做分段 LRU（Segmented LRU）算法，新算法主要是为了更好的利用内存，并提升性能。包含了二个重要的线程，本文先讲 maintainer 线程，后一篇讲 crawler 线程。

每个 Slab-class 有一个 LRU，每个 LRU 又由四个子 LRU 组成，每个子 LRU 维护独立的锁（mutex lock），所有的 LRU 由一个独立的线程维护（这和旧的 LRU 算法有很大的不同），称之为 “LRU maintainer” 线程。

每个 item 有一个 flag，存储在其元数据中，标识其活跃程度：

- FETCHED：如果一个 item 有请求操作，其 flag 等于 FETCHED。
- ACTIVE：如果一个 item 第二次被请求则会标记为 ACTIVE；当一个 item 发生 bump 或被移动了，flag 会被清空。
- INACTIVE：不活跃状态。

这四个子 LRU 包含了四个独立的 queue，相关的 queue 可能会迁移到其他的 queue，这么设计就是为了减少 bump 的产生，先看一张图：

![结构图](memcached1.5-lru-fd-jg.png)

（1）HOT queue：如果一个 item 的过期时间（TTL）很短，会进入该队列，在 HOT queue 中不会发生 bump，如果一个 item 到达了 queue 的 tail，那么会进入到 WARM 队列（如果 item 是 ACTIVE 状态）或者 COLD 队列（如果 item 处于不活跃状态）。

（2）WARM queue：如果一个 item 不是 FETCHED，永远不会进入这个队列，该队列里面的 item TTL 时间相对较长，这个队列的 lock 竞争会很少。该队列 tail 处的一个 item 如果再一次被访问，会 bump 回到 head，否则移动到 COLD 队列。 

（3）COLD queue：包含了最不活跃的 item，一旦该队列内存满了，该队列 tail 处的 item 会被 evict。

如果一个 item 被激活了，那么会异步移动到 WARM 队列，如果某个时间段内大量的 COLD item 被激活了，bump 操作可能会处于满负载，这个时候它会什么也不做（不移动到 WARM queue），避免影响工作线程的性能。

（4）TEMP queue：该队列中的 item TTL 通常只有几秒，该列队中的 item 永远不会发生 bump，也不会进入其他队列，节省了 CPU 时间，也避免了 lock 竞争。

HOT 和 WARM LAU queue 有内存使用的限制，而 COLD 和 TEMP 队列没有内存使用限制，这主要是为了避免一些不经常使用的 item 长期占据在相对活跃的队列中。

总结下 LRU Maintainer 线程的任务：

- 迭代每个子 LRU（每个 Slab-class 有4个），然后查看 tail 部的 item。
- 查看每个子 LRU 的内存限制，必要的时候移出一些。
- 回收 tail 部的过期 item（更多的回收由 LRU Crawler 线程处理，后面会说）。
- 异步处理 COLD queue 的 bump 操作（会产生锁）。

除了让内存使用更有效，分段 LRU 还有一些好处：

- 直接 get 的时候不会产生 bump，这对于工作线程的扩展性有好处，而且也不会产生 lock 等待。
- bump 操作都是异步发生的。
- 写操作扩展性更好，比如不会不会在 set 的时候产生内存回收操作。
- 每个 item 的元数据不会变多，


从 memcached1.5开始，分段 LRU 机制默认是启用的，如果想显式启用，可以运行下列两个命令中的任意一个：

```
$ ./memcached -o modern

$ ./memcached -o lru_maintainer
```

如果 memcached 是 1.4 版本，也可以使用下列命令启动：

```
$ ./memcached -o modern
```

在 memcached 运行的时候，也可以通过内部命令动态调整 LRU 算法，比如：

```
# 使用传统的 LRU 算法 
$ lru mode flat 

# 使用分段 LRU 算法 
$ lru mode segmented 
```
 
如果采用分段 LRU 算法，还有更多的子命令参数可以使用：

```
$ lru tune 10 25 0.1 2.0 
```

其中 10 表示 hot 队列的内存占比不能超过 10%，WARM 队列的内存占比不能超过 25%， HOT 队列 tail age 大于 COLD 队列 tail age 的 10%，WARM 队列的 tail age 是 COLD 队列 tail age 的 2 倍。

至于这些参数的作用，引用下面的解释（更容易理解）：
 
> HOT and WARM LRU’s are limited in size primarily by percentage of memory used, while COLD and TEMP are unlimited. HOT and WARM have a secondary tail age limit, relative to the age of the tail of COLD. This prevents very idle items from persisting in the active queues needlessly.

另外一个配置：

```
$ lru temp_ttl ttl
```

如果 ttl 值小于0，表示禁用 TEMP queue；如果大于 0，那么过期时间小于 TTL 的 item 就会进入 TEMP queue，而且除非 item 过期或删除，否则不会离开该队列。
 

可以通过 stats settings 了解更多分段 LRU 的设置：

| 参数     | 说明  |  
| ------------- |:-------------:| 
| lru_maintainer_thread | 是否启用 maintainer 线程|
| lru_segmented| 是否启用 分段 LRU |
| hot_lru_pct   | HOT LRU 占所有 LRU 内存的百分比 |
| warm_lru_pct     | WARM LRU 占所有 LRU 内存的百分比    |
| hot_max_factor   | HOT 队列 tail age 大于 COLD 队列 tail age 的值|
| warm_max_factor  | WARM 队列 tail age 大于 COLD 队列 tail age 的值|
| temp_lru          | 是否启用 TEMP LRU | 
| temporary_ttl     | 小于该值的 item 进入 TEMP LRU |
 

可以通过 stats 命令了解分段 LRU 的运行数据：
 
| 参数     | 说明  |  
| ------------- |:-------------:| 
| moves_to_cold  | 从 HOT/WARM 移动到 COLD 的 item 个数 |
| moves_to_warm  | 从 COLD 移动到 WARM 的 item 个数 |
| moves_within_lru | HOT or WARM 交换的个数| 
 
也可以输入 stats item 了解详细的分段 LRU 运行数据：

| 参数     | 说明  |  
| ------------- |:-------------:| 
| number_hot  | 目前在 HOT LRU 中的 item 个数 |
| number_warm  | 目前在 WARM LRU 中的 item 个数 |
| number_cold  | 目前在 COLD LRU 中的 item 个数 |
| number_temp  | 目前在 TEMP LRU 中的 item 个数 |
| age_hot | HOT LRU 中最老 item 的时间 | 
| age_warm | WARM LRU 中最老 item 的时间 | 
| age | LRU 中最老 item 的时间 |  

memcached 相关文章：

- [聊聊memcached1.4的LRU算法](https://mp.weixin.qq.com/s/hfXWGm2fuyeThHawEHub-w)
- [简单理解memcached的内存分配](https://mp.weixin.qq.com/s/8fs5YU8drC5vUt1RxOgifw) 
- [使用Memcached实现抽奖活动](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)  
- [php-memcached扩展升级小记](https://mp.weixin.qq.com/s/HZf1GVpl92gL6lqbLwLBww)  
- [再议php-memcached扩展的编译，进一步理解phpize](https://mp.weixin.qq.com/s/v2FvcJd_SDob19ToXxLWIA) 

--- 

欢迎大家关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），所有文章都是原创，主要来源于平时工作中遇到的问题和学习中的一些想法；也可以了解我的书[《深入浅出HTTPS：从原理的实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)