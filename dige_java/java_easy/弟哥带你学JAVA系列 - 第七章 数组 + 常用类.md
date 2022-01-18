## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 数组
    大家帮点个赞吧～ 谢谢啦～
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35701122c4894678b08be7436a87e461~tplv-k3u1fbpfcp-zoom-1.image)

因为这种声明只是生成一个引用，真正的数组是在堆空间中生成的![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/633661b123d04f09acde4c4ade580751~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d216781aac71471e913f71923d4b9cfc~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfc14c8b664747ceb10009e80a34466f~tplv-k3u1fbpfcp-zoom-1.image)




java的数组得先分配空间才能赋值

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8552540295b426e862d13146f4e4856~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b26b400da069470bb803b160194a36aa~tplv-k3u1fbpfcp-zoom-1.image)




数组存储检查是很严格的，它只能存储**创建时元素类型**

```
Pair[] table = new Pair[10];
table[0] = new Object(); //编译错误
```

很明显直接存储，编译器会报错。那么将数组向上转换一下，再存储呢？

```
Pair[] table = new Pair[10];
Object[] o = table; //自动转换
o[0] = new Object();
```

此时编译器是不会报错的，但是运行时会抛出**ArrayStoreException**异常

Arraycopy

注：<https://blog.csdn.net/weixin_30990711/article/details/114998641>

Arraycopy是把数组里的内容生成一个地址不一，内容一致的副本赋给另一个数组，所以对于一维数组，原数组改变后不影响另一个数组的值，而多维数组是将引用生成副本后赋值给另一个数组，故原数组修改内容时另一个数组也会变动

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/333d4a1df9f64265ab6e1ac3f2f09acc~tplv-k3u1fbpfcp-zoom-1.image)

# String

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3fa2b3cd27b45c6bc24ccebe8f39aac~tplv-k3u1fbpfcp-zoom-1.image)



## StringBuffer

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a79370ee70864d57adbd032b500feabb~tplv-k3u1fbpfcp-zoom-1.image)

<https://blog.csdn.net/u011702479/article/details/82262823>

String.format() <https://blog.csdn.net/anita9999/article/details/82346552>

# 包装类

Integer Double Float等等

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b566c2d3f654a2d9f4fb32fb1d3c6a0~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fbfdb619c944541b28e09fb55516757~tplv-k3u1fbpfcp-zoom-1.image)



# Math类

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b95b13e06eb4b6ca67d33f9ed9a2867~tplv-k3u1fbpfcp-zoom-1.image)



# File类

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c55756f1c156438a996c6df9aa189a47~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4686afe92a024e189454a3cd9a43b5c6~tplv-k3u1fbpfcp-zoom-1.image)




# 枚举

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab102a547b5b405684b99d38a118b0c8~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/863719991fc04445b3f3e6ccc7ed9dce~tplv-k3u1fbpfcp-zoom-1.image)

