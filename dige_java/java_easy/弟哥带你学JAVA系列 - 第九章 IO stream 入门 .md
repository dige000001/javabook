## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 基本概念

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b98f2677cbf240cc89750618acf6504b~tplv-k3u1fbpfcp-zoom-1.image)

节点流：文件到程序直接传输

处理流：文件到程序过程中有其他处理 即流上流

## InputStream

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb12f7c672ac46c8a914f8dac1142421~tplv-k3u1fbpfcp-zoom-1.image)

# ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9251e665ddf648609a2142fae2488abc~tplv-k3u1fbpfcp-zoom-1.image)

## OutputStream

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7289962033da4d13ae29e82dca281e6d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fd9b2df5aec4bcdb2df791e5b8ee6fa~tplv-k3u1fbpfcp-zoom-1.image)

## Reader

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1b75ff3ea24444e8637d3aeeac35867~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/832d2c2f939343f9864550dcbb080cdc~tplv-k3u1fbpfcp-zoom-1.image)

## Writer

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92da0a4772764cecbb037bab195c57f5~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-


# 二

## 节点流

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ae4e49009ab4b4a9517f8c1ef7246c9~tplv-k3u1fbpfcp-zoom-1.image)



示例：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89387e554da041db90987a2f6edf4766~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb56725769db457b9704cdfc096f2e0f~tplv-k3u1fbpfcp-zoom-1.image)

## 处理流

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd233604a7f14a4d8269f4ee10363ac4~tplv-k3u1fbpfcp-zoom-1.image)

### 缓冲流

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c1e413da4164a5da908e105a459d4c7~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e8b4f90726f487a842a09046d1a0349~tplv-k3u1fbpfcp-zoom-1.image)

### 转换流

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/907adf098c764a579141c29078d2796d~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c5f6d5643ac4e729d532ede4b5ec484~tplv-k3u1fbpfcp-zoom-1.image)

### 数据流

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/456a0914dfac4fe7bbb01c2349050bd2~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/854ddf60fb1c4b7faf0748ace4737eb6~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4c8064f50ac44c3a1f524243628c6cb~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/460f1ff9b54d467ba56c1d033feb0f36~tplv-k3u1fbpfcp-zoom-1.image)

输出至文件

### Object流




Serializable接口：用于将对象序列化，方便存储于硬盘或网络传输

Class T implement Serializable { ……)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac0fc7f4287a46eb9f5ca8322dc485a6~tplv-k3u1fbpfcp-zoom-1.image)

注：transient 修饰的成员变量在序列化时会被忽略

# 三

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/062b2bc7d0044001a7c29f4fbdcc4c0d~tplv-k3u1fbpfcp-zoom-1.image)

