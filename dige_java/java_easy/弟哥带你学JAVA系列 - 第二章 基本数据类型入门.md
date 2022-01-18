## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

## Part1 基础数据类型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6aaf938dea884c42bad2e71646a7cea1~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b971d296afc444a3a0772f248d141546~tplv-k3u1fbpfcp-zoom-1.image)

采用的是UTF-16

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/021c54a9e2404a819de326667a1f0477~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a465616702d4cdb9a65a12bc3ef6fcc~tplv-k3u1fbpfcp-zoom-1.image)




## Part2 数据类型转换

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a462fb3d2b0241d8b3bdf543964376f6~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/15f5049fb8bd4ac680649e88b8f0d1a2~tplv-k3u1fbpfcp-zoom-1.image)




<https://blog.csdn.net/weixin_39890289/article/details/114055762>

**示例：**

char a = 97;                                 -->为char类型变量 a 赋值常量值 97。

char b = 'a'+3;                             -->d               // 97+3=100，ASCII对应的字符为 d。

char c = a+3;                               -->报错        //无法从int类型转换为char类型，接下来让我们了解下为什么会不

//能这样运算：

首先，我们先知道在jvm内存机制中，char类型数据运算是将字符在ASCII表对应的整数以int类型参与运算(可以认为' a '=97)，常量(97)与常量(3)运算得到一个新的常量(100)，**常量赋值给变量(b)，不存在强制转换，只要这个接受变量(b)的类型范围大于这个常量即可**。**而变量声明时需要定义数据类型**(例：char a)，内存就为这个变量划分一个char类型大小的空间，其中变量(a)的值是可变的，而常量(3)的值是不变的，两个运算得到的还是一个变量，本例中(a+3)是int类型的变量，**而int类型变量(a+3)赋值给char类型变量(c)需要强制转换，因此会报错**。

[\
\
](https://blog.csdn.net/weixin_39890289/article/details/114055762)

**而对于float，double常量赋值float也需要强转，因为不能通过直接去掉高位的方法自动转换。**




练习：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fea09acd4e64a7aa48db7503a2ea270~tplv-k3u1fbpfcp-zoom-1.image)

答案

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eee2cfa7d9564426a1194a4cf0b80dac~tplv-k3u1fbpfcp-zoom-1.image)




## Part3 基础语法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abc6ec2520e9431e89e3f5faac2c6831~tplv-k3u1fbpfcp-zoom-1.image)

