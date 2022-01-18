## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

**一、含义**

事务：一条或多条sql语句组成一个执行单位，一组sql语句要么都执行要么都不执行

**二、特点（ACID）**

A 原子性：一个事务是不可再分割的整体，要么都执行要么都不执行

C 一致性：一个事务可以使数据从一个一致状态切换到另外一个一致的状态

I 隔离性：一个事务不受其他事务的干扰，多个事务互相隔离的

D 持久性：一个事务一旦提交了，则永久的持久化到本地



**三、事务的使用步骤 ★**

了解：

隐式（自动）事务：没有明显的开启和结束，本身就是一条事务可以自动提交，比如insert、update、delete

**显式事务：具有明显的开启和结束**


使用显式事务：

①开启事务

set autocommit=0;

start transaction;#可以省略




②编写一组逻辑sql语句

注意：sql语句支持的是insert、update、delete




设置回滚点：

savepoint 回滚点名;




③结束事务

提交：commit;

回滚：rollback;

回滚到指定的地方：rollback to 回滚点名;

1、不管autocommit 是1还是0 \
START TRANSACTION 后，只有当commit数据才会生效，ROLLBACK后就会回滚。

2、当autocommit 为 0 时\
不管有没有START TRANSACTION。\
只有当commit数据才会生效，ROLLBACK后就会回滚。




**#1.演示事务的使用步骤**

#开启事务

SET autocommit=0;

START TRANSACTION;

#编写一组事务的语句

UPDATE account SET balance = 1000 WHERE username='张无忌';

UPDATE account SET balance = 1000 WHERE username='赵敏';




#结束事务

COMMIT;




**#2.演示事务对于delete和truncate的处理的区别**


delete语句是dml,这个操作会放到rollback segement中,事务提交之后才生效;如果有相应的trigger,执行的时候将被触发. \
truncate,drop是ddl, **操作立即生效,** 原数据不放到rollback segment中,不能回滚. 操作不触发trigger. 



**#3.演示savepoint 的使用**

SET autocommit=0;

START TRANSACTION;

DELETE FROM account WHERE id=25;

SAVEPOINT a;#设置保存点

DELETE FROM account WHERE id=28;

ROLLBACK TO a;#回滚到保存点




SELECT * FROM account;




四、并发事务

1、事务的并发问题是如何发生的？

多个事务 同时 操作 同一个数据库的相同数据时

2、并发问题都有哪些？

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f50a117da92e432f8c3e4269a4175b9b~tplv-k3u1fbpfcp-zoom-1.image)

3、如何解决并发问题

通过设置隔离级别来解决并发问题

4、隔离级别

脏读 不可重复读 幻读

read uncommitted:读未提交 × × × 

read committed：读已提交 √ × ×

repeatable read：可重复读 √ √ ×

serializable：串行化 √ √ √



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94045d35c7424839a4868a9716be59b4~tplv-k3u1fbpfcp-zoom-1.image)





