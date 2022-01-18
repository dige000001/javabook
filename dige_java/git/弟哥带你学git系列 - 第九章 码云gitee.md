---
公众号：弟哥带你进大厂
---
# 第 9 章 国内代码托管中心-码云




## gitee 简介

众所周知，GitHub 服务器在国外，使用 GitHub 作为项目托管网站，如果网速不好的话，严重影响使用体验，甚至会出现登录不上的情况。针对这个情况，大家也可以使用国内的项目托管网站-码云。

码云是开源中国推出的基于 Git 的代码托管服务中心，网址是 <https://gitee.com/> ，使用方式跟 GitHub 一样，而且它还是一个中文网站，如果你英文不是很好它是最好的选择。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d444f8f4a85d499483a3fa4448a4809b~tplv-k3u1fbpfcp-zoom-1.image)

##  码云帐号注册和登录

进入码云官网地址：<https://gitee.com/>，点击注册 Gitee




输入个人信息，进行注册即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c487f5614044b578cc81c05f4caa375~tplv-k3u1fbpfcp-zoom-1.image)

帐号注册成功以后，直接登录。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c60e33d5e70e4188aa6a3580663298e6~tplv-k3u1fbpfcp-zoom-1.image)

登录以后，就可以看到码云官网首页了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a64a4b55d934b258e6e15470c373d80~tplv-k3u1fbpfcp-zoom-1.image)

## 码云创建远程库

点击首页右上角的加号，选择下面的新建仓库



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efad8ed72f6941588a0e5008de42e723~tplv-k3u1fbpfcp-zoom-1.image)




填写仓库名称，路径和选择是否开源（共开库或私有库）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e25352453e03475a9de9304f4fafc960~tplv-k3u1fbpfcp-zoom-1.image)

最后根据需求选择分支模型，然后点击创建按钮。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4eace4da06fc4d2babbf3890b3248a66~tplv-k3u1fbpfcp-zoom-1.image)

远程库创建好以后，就可以看到 HTTPS 和 SSH 的链接。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcf016e093cb480f96b5422c0942ef56~tplv-k3u1fbpfcp-zoom-1.image)

## IDEA 集成码云

### IDEA 安装码云插件

Idea 默认不带码云插件，我们第一步要安装Gitee 插件。



如图所示，在 Idea 插件商店搜索 Gitee，然后点击右侧的 Install 按钮。







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/866e21275f054fce996b7039e846d8e8~tplv-k3u1fbpfcp-zoom-1.image)

Idea 链接码云和链接GitHub 几乎一样，安装成功后，重启 Idea。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e1d4b4d77a974dd59db104743acf2502~tplv-k3u1fbpfcp-zoom-1.image)

Idea 重启以后在 Version Control 设置里面看到Gitee，说明码云插件安装成功。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab3593f4734b496b8caf271b89a16e44~tplv-k3u1fbpfcp-zoom-1.image)

然后在码云插件里面添加码云帐号，我们就可以用 Idea 连接码云了。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25127097d55440b491a7f65677221bdc~tplv-k3u1fbpfcp-zoom-1.image)



## IDEA 连接码云

Idea 连接码云和连接 GitHub 几乎一样，首先在 Idea 里面创建一个工程，初始化 git 工程，然后将代码添加到暂存区，提交到本地库，这些步骤上面已经讲过，此处不再赘述。

## 将本地代码 push 到码云远程库




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c9737a18333443783daa8046cdea38a~tplv-k3u1fbpfcp-zoom-1.image)

自定义远程库链接。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/717fe52669ed4559907b36f61333c291~tplv-k3u1fbpfcp-zoom-1.image)




给远程库链接定义个 name，然后再 URL 里面填入码云远程库的 HTTPS 链接即可。码



云服务器在国内，用HTTPS 链接即可，没必要用SSH 免密链接。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b491a52cba2e45b8a48b4446ab958533~tplv-k3u1fbpfcp-zoom-1.image)



然后选择定义好的远程链接，点击 Push 即可。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd0853557cb74329870b03a555e89bf7~tplv-k3u1fbpfcp-zoom-1.image)

看到提示就说明 Push 远程库成功。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfa2abc8a2324d8b9ea545a4fc0f43b8~tplv-k3u1fbpfcp-zoom-1.image)

去码云远程库查看代码。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96cc5a52dc9142abacaa964b33230de0~tplv-k3u1fbpfcp-zoom-1.image)

只要码云远程库链接定义好以后，对码云远程库进行 pull 和 clone 的操作和 Github 一致，此处不再赘述。

## 码云复制 GitHub 项目

码云提供了直接复制 GitHub 项目的功能，方便我们做项目的迁移和下载。具体操作如下：




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/855d14404d514aedb1abd02d0734d431~tplv-k3u1fbpfcp-zoom-1.image)

将 GitHub 的远程库HTTPS 链接复制过来，点击创建按钮即可。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/47c4f8ec718341bbbc5f4946a72aa629~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6feac8d1616a460c941dfad547ae0a53~tplv-k3u1fbpfcp-zoom-1.image)



如果GitHub 项目更新了以后，在码云项目端可以手动重新同步，进行更新！




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0bb025b6b714ab683246457b2c88798~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1a4c32612ee5494b87cac0bb1ffd453e~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ece0063442c4c57839ad721d7c48bd8~tplv-k3u1fbpfcp-zoom-1.image)

