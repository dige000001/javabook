## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 异常的概念以及分类

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ea3553bbb3b840858ce37b810810b0e1~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d3fa1849b8f463c9dc7fe23c7b42489~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cae4c67f12b429ca4b4d3eecf33de27~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c3709afa15a749a19312978483051f7d~tplv-k3u1fbpfcp-zoom-1.image)

所以我们一般只抛出Exception异常，不抛出RuntimeException

#

# 声明并抛出异常

注：声明有可能抛出的异常时，在使用时一定要用try catch语句处理，如果无法处理，那就抛出

通常应该捕获那些知道如何处理的异常，对于不知道如何处理的异常应继续抛出

若A方法调用B方法，B方法有可能抛出异常，则A有两种办法：1.try catch 2.声明A方法时抛出和B方法一样的异常(或其异常的父类异常)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a2d6411c3f54e8fa12c09ccd30f140b~tplv-k3u1fbpfcp-zoom-1.image)

## 自定义异常：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8daad01366ed461da89908a78e5fa96e~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0acce6d3bfc040a99760197b75384468~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8f403287e2b4ba9ad184d213d3edd0b~tplv-k3u1fbpfcp-zoom-1.image)



## 子类覆盖父类方法

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/309bc73864be4c8a80641b66f1cf90bd~tplv-k3u1fbpfcp-zoom-1.image)




# 异常的捕获和处理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/974689f2a4644996aa66b5703d8c8014~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/986dbe4a9b9c4d92883f0d96d1ddf740~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3563f368abb4ce8abf121e3421fe709~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed49526b5e7a4037a1aac05691296287~tplv-k3u1fbpfcp-zoom-1.image)




## 捕获多个异常并分别处理：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09adbd78abe1459393ebe4b34494354c~tplv-k3u1fbpfcp-zoom-1.image)

## 再次抛出异常与异常链

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8eb8f7db2244494ba11c5301292caca1~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85078342f43d4f3fad688e2ac764770d~tplv-k3u1fbpfcp-zoom-1.image)

一些细节：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf233aac8cfd47e98402f81211834da2~tplv-k3u1fbpfcp-zoom-1.image)

## finally

注：有finally字句时，程序在执行完finally之后才抛出异常，因此若finally中的语句也可能抛出异常，则会覆盖原异常

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36423f31b47545fab9bacccbde5a6810~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5cc66d2168ec4d3ea8969ba3bedc13f2~tplv-k3u1fbpfcp-zoom-1.image)

## java中异常抛出后代码还会继续执行吗

```
//代码1
public static void test() throws Exception  {

    throw new Exception("参数越界"); 
    System.out.println("异常后"); //编译错误，「无法访问的语句」
}

//代码2
try{
    throw new Exception("参数越界"); 
}catch(Exception e) {
    e.printStackTrace();
}
System.out.println("异常后");//可以执行

//代码3
if(true) {
    throw new Exception("参数越界"); 
}
System.out.println("异常后"); //抛出异常，不会执行
```

1.  若一段代码前有异常抛出，并且这个异常没有被捕获，这段代码将产生编译时错误「无法访问的语句」。如代码1
1.  若一段代码前有异常抛出，并且这个异常被try...catch所捕获，若此时catch语句中没有抛出新的异常，则这段代码能够被执行，否则，同第1条。如代码2

<!---->

3.  若在一个条件语句中抛出异常，则程序能被编译，但后面的语句不会被执行。如代码3



## 带资源的try语句

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a5d2388bc7941319ece76830b437de5~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49de2a6b19c4487eac7409a54676216a~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64c8e74d96f74c42a36ff38ec063986b~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ff7e8e41dc24f1ebd22951a656e76e7~tplv-k3u1fbpfcp-zoom-1.image)