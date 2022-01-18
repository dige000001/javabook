## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 事务的基本操作

### 开启事务

multi

作用：设定事务的开启位置，此指令执行后，后续的所有指令均加入到事务中

### 执行事务

exec

作用：设定事务的结束位置，同时执行事务。与multi成对出现，成对使用

注意：**加入事务的命令暂时进入到任务队列中，并没有立即执行，只有执行exec命令才开始执行**

### 取消事务

discard

作用：终止当前事务的定义，发生在multi之后，exec之前

# 事务的工作流程

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75637628f6614061b2e2cd9ebdfa9ee8~tplv-k3u1fbpfcp-zoom-1.image)

# 事务的注意事项

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4174bcb6a504239841cb250447f7794~tplv-k3u1fbpfcp-zoom-1.image)

### 如何手动回滚

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82640a8182f44657bcad886f78f6a5ab~tplv-k3u1fbpfcp-zoom-1.image)


# 锁

## 基于特定条件的事务执行——普通锁

对 key添加监视锁，在执行exec前如果key发生了变化，终止事务执行

watch key1 [ key2......]

取消对所有key的监视

unwatch

## 基于特定条件的事务执行——分布式锁

**使用setnx设置一个公共锁**

setnx lock-key value

利用setnx命令的返回值特征，有值则返回设置失败，无值则返回设置成功

-   对于返回设置成功的，拥有控制权，进行下一步的具体业务操作
-   对于返回设置失败的，不具有控制权，排队或等待




操作完毕通过del操作释放锁

注意:上述解决方案是一种设计概念，依赖规范保障，具有风险性

### 分布式锁改良

**使用expire为锁key添加时间限定，到时不释放，放弃锁**

expire lock-key second

pexpire lock-key milliseconds

由于操作通常都是微秒或毫秒级，因此该锁定时间不宜设置过大。具体时间需要业务测试后确认。

-   例如:持有锁的操作最长执行时间127ms，最短执行时间7ms。
-   测试百万次最长执行时间对应命令的最大耗时，测试百万次网络延迟平均耗时

<!---->

-   锁时间设定推荐:最大耗时*120%+平均网络延迟*110%
-   如果业务最大耗时<<网络平均延迟，通常为2个数量级，取其中单个耗时较长即可


