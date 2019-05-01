本篇文章承接[memcached1.5更好的LRU算法，了解下maintainer线程](https://mp.weixin.qq.com/s/BG3wpLOWQJrKd0_btxo1Tw)，如果还没有阅读，建议先读一下。虽然 LRU Maintainer 解决了很多问题，但结合 Memcached 内存分配机制，它还有一些潜在的问题，比如说很难动态调整内存的大小；再比如某些 Slab-class 可能存储了很少数量的 item（和 item 的大小有关系）；再比如一个空间很大但已经过期的 item 其实可以存储几百个小空间 item；还有 LRU Maintainer 并没有过期 item 回收的功能。

为了解决这些问题，memcached1.5 版本引进了 LRU crawler, 它是一个异步的后台线程，扫描 LRU 中的所有 item，然后回收过期 item，**或者检查整个 Slab-class，进行相应的调整（这部分说的不多，主要说回收）**。

crawler 在每个 Slab-class 的每个子 LRU 的 tail 部插入一个特别的 crawler items，然后从子 LRU 的 tail 到 head 不断进行扫描，如果发现有过期的 item，就进行回收。

它在给每个子 LRU 进行扫描的时候，会构建一个直方图，通过直方图决定下一次扫描的时间，举个例子：

- 假如 Slab-class 1 有 100 万个 item，过期时间都是 0（也就是不过期），那么最多每小时扫描一次（因为再扫描也回收不了多少内存）。
- 假如 Slab-class 5 有 10万个 item，其中 1% 的 item 5分钟后过期，那么 crawler 将智能的在五分钟后再一次扫描，因为能够回收很多内存。
 
crawler 还有很多的智能调度策略，比如 Slab-class 越高，代表存储的单个 item 空间更大，尽快回收能够释放更多的内存。

结合分段 LRU 机制，crawler 也有很多调度策略，比如 HOT queue 如果有很多 item （TTL 较短），那么应该频繁的扫描，同时避免频繁扫描 COLD queue。

这些调度策略都是为了减少不必要的 crawler 工作。

通过内部命令可以调整相应参数：

```
# 开启或禁止 crawler
lru_crawler <enable|disable>

# 设置 crawler 多久运行一次
lru_crawler sleep <microseconds>

# 手动给编号为 classid 的 Slab-class 进行 crawler 
lru_crawler crawl <classid,classid,classid|all> 
```

介绍一个比较有用的 crawler 命令，格式如下：

```
lru_crawler metadump <classid,classid,classid|all>
```

举个例子：

```
lru_crawler metadump 1,all 
key=x exp=-1 la=1545878079 cas=1 fetch=yes cls=1 size=61
key=xx exp=-1 la=1546066251 cas=2 fetch=no cls=1 size=63
```

metadump 命令和 cachedump 命令很类似，但不会有锁操作，返回 LRU 中的所有 item，包括过期时间，上次存取时间等等。

运行 stats 命令，查看 LRU crawler 相关数据：

| 参数     | 说明  |  
| ------------- |:-------------:| 
| number_hot  | 目前在 HOT LRU 中的 item 个数 |
| slab_reassign_running | 目前是否有 slab page 移动| 
| slabs_moved | 总共有多少 slab page 被迁移 |
| crawler_reclaimed     | 被 crawler 回收的 item 总个数 |
| crawler_items_checked | 总共有多少个 item 被 crawler 检查 | 

到目前位置，Memcached 1.5 所有的 LRU 算法机制已经讲完，理解他们对于掌握 Memcached 很有好处，后面会从应用的角度理解 Memcached，敬请关注。

**memcached 相关文章：**

- [memcached1.5更好的LRU算法，了解下maintainer线程](https://mp.weixin.qq.com/s/BG3wpLOWQJrKd0_btxo1Tw) 
- [聊聊memcached1.4的LRU算法](https://mp.weixin.qq.com/s/hfXWGm2fuyeThHawEHub-w)
- [简单理解memcached的内存分配](https://mp.weixin.qq.com/s/8fs5YU8drC5vUt1RxOgifw) 
- [使用Memcached实现抽奖活动](https://mp.weixin.qq.com/s/agUU5ZjcVep-vPIKVjB6Fg)  
--- 

欢迎大家关注我的公众号（ID：yudadanwx，虞大胆的叽叽喳喳），所有文章都是原创，主要来源于平时工作中遇到的问题和学习中的一些想法；也可以了解我的书[《深入浅出HTTPS：从原理的实战》](https://mp.weixin.qq.com/s/9KpVnHc3yWfy1Qwal_HISA)