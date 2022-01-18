## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# String

## 用法：

### 普通赋值与取值

set key value

get key

### 多指令赋值与取值

mset key1 value1 key2 value2 

mget key1 key2

### 附加

append key value //若key存在，则附加，若不存在，则新建

### 数值增减操作

incr key //给key的value加1

incrby key increment //给key的value加increment的值，要求为整数型

incrbyfloat key increment //给key的value加increment的值，要求为浮点数

decr key

decrby key increment

### 设置数据具有指定的生命周期

setex key seconds value

psetex key milliseconds value

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e72d3888bc7447df81ae61030d662182~tplv-k3u1fbpfcp-zoom-1.image)

## 注意事项



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/747242703e0846b4b11e4e25b70c65be~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcbd114fa94044dda640f92acf306829~tplv-k3u1fbpfcp-zoom-1.image)

# Hash

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1363298bc9f4cac8033c14f4f32bbb4~tplv-k3u1fbpfcp-zoom-1.image)

## 使用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f25ec5ce2b584fecbbf6e06712a2d77e~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34c9748388454c2d8e9dd5ecc18479db~tplv-k3u1fbpfcp-zoom-1.image)若field存在，则不变，否则新建

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa37258b9b8b4abe88aeffddab0af25a~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63b80fc0e6dc407c82d5a53cdc28d0fb~tplv-k3u1fbpfcp-zoom-1.image)

## 注意事项

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8bd4d426b0a4aedb8222298c479c40d~tplv-k3u1fbpfcp-zoom-1.image)

# List

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/679cf40ea23c4999b1f41404c1f9cb4d~tplv-k3u1fbpfcp-zoom-1.image)

## 使用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6681298ec28948afacdb36b02581f1bc~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1309f0932aeb409b809736c99e0d67bb~tplv-k3u1fbpfcp-zoom-1.image)

lrange key 0 -1 表示取所有值

lrange key 0 -2 表示取到倒数第二个值

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ca7514ec2e245eab7fdfeef22692841~tplv-k3u1fbpfcp-zoom-1.image)

[单线程的redis如何处理阻塞命令](https://blog.csdn.net/u013041642/article/details/98606804)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f741e88a1e4841d2869c7899c3aed831~tplv-k3u1fbpfcp-zoom-1.image)

对应的，有rrem

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85e5a473e80243459b2ee1a21d329b0c~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/51d43c8b382f4adfad204b8e2d2fca9d~tplv-k3u1fbpfcp-zoom-1.image)

可以看到，移除了前3个'a'

# Set

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a5cb6b1b93140858c974490698b362a~tplv-k3u1fbpfcp-zoom-1.image)

## 使用

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82bf49b9b1924cfda90e8d4c8a213b3e~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6728c103ba6408484a8858edf950a41~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9328ba052bfb49df95abc513f59e67ba~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5090300f33144b5fb7c5f99ecd0259cd~tplv-k3u1fbpfcp-zoom-1.image)

# sorted_set

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb42b7a3d77649a9b4490866e8266ed3~tplv-k3u1fbpfcp-zoom-1.image)

## 操作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24ca0d57b4b7485b91c8306a7f87fb7a~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5e64c7b7edd456aa86d3873e7030a6b~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27d64e20b5a14029a8fe18a6193a079f~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e841d730ee14734b83fc53fcc761d3e~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9d4cf4c65b874f6785a1140e2915768d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c8163b4841774f2686e78c51c9c1d7ca~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b55985afc8841758a7ae16109619eaa~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f31312e429f844e5aceee17de122f579~tplv-k3u1fbpfcp-zoom-1.image)

## 注意事项

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eb2eaaa93ce4d179beaf281f2879f7b~tplv-k3u1fbpfcp-zoom-1.image)

# bitmaps

获取指定key对应偏移量上的bit值

getbit key offset

设置指定key对应偏移量上的bit值,value只能是1或0

setbit key offset value

对指定key按位进行交、并、非、异或操作，并将结果保存到destKey中

bitop op destKey key1 [ key2...]

-   and：交
-   or：并 

<!---->

-   not：非 
-   xor：异或

统计指定key中1的数量

bitcount key [start end]



