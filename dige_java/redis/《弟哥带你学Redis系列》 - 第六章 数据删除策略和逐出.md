## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 数据删除策略

## 定时删除

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df0b5bc898834bceb501359ba7dbeb3d~tplv-k3u1fbpfcp-zoom-1.image)

## 惰性删除

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0988c1c2c46141d68f49dca6afe710aa~tplv-k3u1fbpfcp-zoom-1.image)

在这种方式下，每次get操作都会和一个名为expiredIfNeeded()的函数绑定

## 定时删除

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43eae770ec8146269d3a71391fdc92d7~tplv-k3u1fbpfcp-zoom-1.image)

周期性轮询redis库中的时效性数据，采用随机抽取的策略，利用过期数据占比的方式控制删除频度

特点1：CPU性能占用设置有峰值，检测频度可自定义设置

特点2：内存压力不是很大，长期占用内存的冷数据会被持续清理

总结：周期性抽查存储空间(随机抽查，重点抽查)

# 逐出

当redis里的数据都没过期，且没有足够的空间存放新数据时，就要采取逐出策略删除一些数据

## 相关配置

****

**最大可使用内存**

maxmemory

占用物理内存的比例，默认值为0，表示不限制。生产环境中根据需求设定，通常设置在50%以上。

****

**每次选取待删除数据的个数**

maxmemory-samples

选取数据时并不会全库扫描，导致严重的性能消耗，降低读写性能。因此采用随机获取数据的方式作为待检测删除数据

****

**删除策略**

maxmemory-policy

达到最大内存后的，对被挑选出来的数据进行删除的策略

### 删除策略


1.  **检测易失数据(可能会过期的数据集server.db[i].expires )**

-   **volatile-Iru**：挑选最近最少使用的数据淘汰
-   **volatile-lfu**：挑选最近使用次数最少的数据淘汰

<!---->

-   **volatile-ttl**：挑选将要过期的数据淘汰
-   **volatile-random**：任意选择数据淘汰

2.  **检测全库数据（所有数据集server.db[i].dict )**

-   **allkeys-Iru**：挑选最近最少使用的数据淘汰
-   **allkeys-lfu**：挑选最近使用次数最少的数据淘汰

<!---->

-   **allkeys-random**：任意选择数据淘汰

3.  **放弃数据驱逐**

-   **no-enviction(驱逐)** ︰禁止驱逐数据（redis4.0中默认策略)，内存满时会引发错误OOM(OutOf Memory)



