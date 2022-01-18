---
公众号：弟哥带你进大厂
---


# 第 7 章 IDEA 集成 Git

## 配置 Git 忽略文件

#### Eclipse 特定文件




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ed7dbef7c364f5181bb83a0a638aa30~tplv-k3u1fbpfcp-zoom-1.image)

#### IDEA 特定文件









![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38faf269daa2471a9af8b48a7b5b1e05~tplv-k3u1fbpfcp-zoom-1.image)



#### Maven 工程的 target 目录




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fb4114dd5de4eb3af9f754ea4a45c5d~tplv-k3u1fbpfcp-zoom-1.image)




#### 问题 1:为什么要忽略他们？

答：与项目的实际功能无关，不参与服务器上部署运行。把它们忽略掉能够屏蔽 IDE 工具之间的差异。

#### 问题 2：怎么忽略？




1.  **创建忽略规则文件** **xxxx.ignore（前缀名随便起，建议是 git.ignore）**




这个文件的存放位置原则上在哪里都可以，为了便于让~/.gitconfig 文件引用，建议也放在用户家目录下

git.ignore 文件模版内容如下：

```
# Compiled class file
*.class
    
# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/
    
# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

# virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
.classpath
.project
.settings
target
.idea
*.iml
```

2.  **在.gitconfig 文件中引用忽略配置文件（此文件在Windows 的家目录中）**

```
[user]
name = Layne
email = Layne@atguigu.com [core]
excludesfile = C:/Users/asus/git.ignore
```

注意：这里要使用“正斜线（/）”，不要使用“反斜线（\）”




## 定位 Git 程序




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d0d73603de34e6782e20c54c3b2e6c3~tplv-k3u1fbpfcp-zoom-1.image)

### 初始化本地库




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85937680012d433e96aeac7f76025f8d~tplv-k3u1fbpfcp-zoom-1.image)

选择要创建Git 本地仓库的工程。









![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5dc721ce0704a1c8eaff1a1b5a92387~tplv-k3u1fbpfcp-zoom-1.image)




## 添加到暂存区

右键点击项目选择Git -> Add 将项目添加到暂存区。







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2c37d16002c4dd49cb3881c174d36f0~tplv-k3u1fbpfcp-zoom-1.image)




## 提交到本地库



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/316337f5f3cf4bcda085afd5c76c1a2e~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53a4567c6d1348528fa5d90f83b2bc55~tplv-k3u1fbpfcp-zoom-1.image)




## 切换版本

在 IDEA 的左下角，点击 Version Control，然后点击Log 查看版本

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac5708a262d644beabe4c0f94119c6e8~tplv-k3u1fbpfcp-zoom-1.image)

右键选择要切换的版本，然后在菜单里点击Checkout Revision。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65059e1855794417b50852d90edd7dc8~tplv-k3u1fbpfcp-zoom-1.image)




## 创建分支

选择Git，在Repository 里面，点击 Branches 按钮。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bd7fb3c697740199454373c5312316a~tplv-k3u1fbpfcp-zoom-1.image)

在弹出的Git Branches 框里，点击 New Branch 按钮。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/297761476ede4fb781e98d1b3acbf7ae~tplv-k3u1fbpfcp-zoom-1.image)




填写分支名称，创建 hot-fix 分支。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fcbcfd5dd2b4a0a825bd08f06975f7e~tplv-k3u1fbpfcp-zoom-1.image)




然后再 IDEA 的右下角看到 hot-fix，说明分支创建成功，并且当前已经切换成 hot-fix 分

支




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d23438c7cd154bf5876a41ad7436f601~tplv-k3u1fbpfcp-zoom-1.image)




## 切换分支

在 IDEA 窗口的右下角，切换到 master 分支。





![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe100c0c64f2427297e97754188bc009~tplv-k3u1fbpfcp-zoom-1.image)




然后在 IDEA 窗口的右下角看到了 master，说明 master 分支切换成功。

\


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/679054e427934f0b8a83608e25e632d1~tplv-k3u1fbpfcp-zoom-1.image)




## 合并分支

在 IDEA 窗口的右下角，将 hot-fix 分支合并到当前master 分支。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42ed07cc101e442791d330215d14e74a~tplv-k3u1fbpfcp-zoom-1.image)

如果代码没有冲突，分支直接合并成功，分支合并成功以后，代码自动提交，无需手动提交本地库。







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b7627dd38f44031819f9a88942cf46f~tplv-k3u1fbpfcp-zoom-1.image)




## 解决冲突

如图所示，如果 master 分支和 hot-fix 分支都修改了代码，在合并分支的时候就会发生冲突。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d3af25aa7ab43a1b40496c2048bb27e~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9dc976bc5483498b8ae6b986c38b1fe4~tplv-k3u1fbpfcp-zoom-1.image)




我们现在站在 master 分支上合并hot-fix 分支，就会发生代码冲突。


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be1e190d366b4334a26dfded9f3a6ce2~tplv-k3u1fbpfcp-zoom-1.image)

点击Conflicts 框里的 Merge 按钮，进行手动合并代码。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc38e281f84a4d5392df64ea1740349c~tplv-k3u1fbpfcp-zoom-1.image)



手动合并完代码以后，点击右下角的 Apply 按钮。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed2c57cf65c546858686b2f2642e9788~tplv-k3u1fbpfcp-zoom-1.image)


代码冲突解决，自动提交本地库。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/006d7033133e4f2a8398512b1ba11b3b~tplv-k3u1fbpfcp-zoom-1.image)




# 第 8 章 IDEA 集成 GitHub




## 设置 GitHub 账号



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c4e5430011f340f5bd343aa6d2b448ee~tplv-k3u1fbpfcp-zoom-1.image)

如果出现 401 等情况连接不上的，是因为网络原因，可以使用以下方式连接：




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59fc777548d74520aa2b00104812ee3d~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f066c8f8f3544042809c6780190eec7e~tplv-k3u1fbpfcp-zoom-1.image)




然后去 GitHub 账户上设置 token。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23683d7132ef462780eeba464ccc78b3~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eefdbdb6f5764967968c7a281d2c9f46~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e77c850dd0d4b11ab10a582bafd8186~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03b141db2be54aab8ff7ae6f5dda2f82~tplv-k3u1fbpfcp-zoom-1.image)








![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cc333f7ed374b10aa989d5bb6787242~tplv-k3u1fbpfcp-zoom-1.image)




点击生成token。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a24ebe6f7344f8a8ea564472ffc4b92~tplv-k3u1fbpfcp-zoom-1.image)

复制红框中的字符串到idea 中。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c5fa01168304682b3aa916d62f48d22~tplv-k3u1fbpfcp-zoom-1.image)

点击登录。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfeefd2cae344923a7888dc1899c679b~tplv-k3u1fbpfcp-zoom-1.image)




## 分享工程到 GitHub




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d6dc71c32384b4088d2c95f57de8726~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b3fcde6053c4f9dbdcb3423c4922ad9~tplv-k3u1fbpfcp-zoom-1.image)







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc1b0a60b024459c965d55b8b86c16cf~tplv-k3u1fbpfcp-zoom-1.image)




来到GitHub 中发现已经帮我们创建好了 gitTest 的远程仓库。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b83d5f7d7724e5dad6281c73a42bbc6~tplv-k3u1fbpfcp-zoom-1.image)




## push 推送本地库到远程库

右键点击项目，可以将当前分支的内容 push 到 GitHub 的远程仓库中。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2eba4f8bf95f4cc393cd084828f716d7~tplv-k3u1fbpfcp-zoom-1.image)










![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03f3765ca66a41208174d214821a53b5~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2197b1adb18a46a6931223c394ca504b~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0373cf81d99d442e8c603846fd57e96f~tplv-k3u1fbpfcp-zoom-1.image)







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e690b5aae63f42f4b41c6c87f034c3ba~tplv-k3u1fbpfcp-zoom-1.image)




注意：push 是将本地库代码推送到远程库，如果本地库代码跟远程库代码版本不一致， push 的操作是会被拒绝的。也就是说，要想 push 成功，一定要保证本地库的版本要比远程库的版本高！因此一个成熟的程序员在动手改本地代码之前，一定会先检查下远程库跟本地代码的区别！如果本地的代码版本已经落后，切记要先 pull 拉取一下远程库的代码，将本地代码更新到最新以后，然后再修改，提交，推送！


## pull 拉取远程库到本地库

右键点击项目，可以将远程仓库的内容 pull 到本地仓库。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84323006395a40e9800a6c8a3076bc69~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/25b7072edb48458fb6f032fc22f7d0b1~tplv-k3u1fbpfcp-zoom-1.image)



注意：pull 是拉取远端仓库代码到本地，如果远程库代码和本地库代码不一致，会自动合并，如果自动合并失败，还会涉及到手动解决冲突的问题。




## clone 克隆远程库到本地




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3cf101f57426485f8aa7d1163c30a147~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e8d4d5a52b947519f3d5c99feaa125b~tplv-k3u1fbpfcp-zoom-1.image)




为 clone 下来的项目创建一个工程，然后点击 Next。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8263fe42e8a41abb34a07b5fee76214~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8453adecea06418da2a2f0f2dd827dc2~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b17b2f44d7bc4ccf9359d7faf3712f3b~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82f19e3cbcc7422291d6569c37a7f6a9~tplv-k3u1fbpfcp-zoom-1.image)

