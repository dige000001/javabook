---
公众号: 弟哥带你进大厂
---
# 第 5 章 Git 团队协作机制

## 团队内协作

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/592b043bbcb249dca6e86883bb50e4b8~tplv-k3u1fbpfcp-zoom-1.image)

## 跨团队协作 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b152dd472bd4b30b01b90baa78f60bf~tplv-k3u1fbpfcp-zoom-1.image)

# 第 6 章 GitHub 操作

GitHub 网址：<https://github.com/>




| 账号                 | 姓名   | 验证邮箱                           |
| ------------------ | ---- | ------------------------------ |
| atguiguyueyue      | 岳不群  | [atguiguyueyue@aliyun.com]()   |
| atguigulinghuchong | 令狐冲  | [atguigulinghuchong@163.com]() |
| atguigudongfang1   | 东方不败 | [atguigudongfang@163.com]()    |

注:此三个账号为讲师使用账号，同学请自行注册，然后三个同学为一组进行团队协作！




## 创建远程仓库




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75916d71976143178eece0ff37e8ef76~tplv-k3u1fbpfcp-zoom-1.image)





![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af2f48f3cb324bfbad86fd34e2afba95~tplv-k3u1fbpfcp-zoom-1.image)




### 远程仓库操作




| **命令名称**               | **作用**                       |
| ---------------------- | ---------------------------- |
| git remote -v          | 查看当前所有远程地址别名                 |
| git remote add 别名 远程地址 | 起别名                          |
| git push 别名 分支         | 推送本地分支上的内容到远程仓库              |
| git clone 远程地址         | 将远程仓库的内容克隆到本地                |
| git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并 |




### 创建远程仓库别名

#### 1）基本语法

**git remote -v 查看当前所有远程地址别名**

**git remote add 别名 远程地址**

**2）案例实操**

Layne@LAPTOP-Layne MINGW64 /d/Git-Space/SH0720 (master)

$ **git remote -v**

Layne@LAPTOP-Layne MINGW64 /d/Git-Space/SH0720 (master)

$ **git remote add ori** [**https://github.com/atguiguyueyue/git-shTest.git**](https://github.com/atguiguyueyue/git-shTest.git)

Layne@LAPTOP-Layne MINGW64 /d/Git-Space/SH0720 (master)

$ **git remote -v**

ori <https://github.com/atguiguyueyue/git-shTest.git> (fetch) 

ori <https://github.com/atguiguyueyue/git-shTest.git> (push)




<https://github.com/atguiguyueyue/git-shTest.git>

**这个地址在创建完远程仓库后生成的连接，如图所示红框中**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a05429e72a98465c828a0d9a7dd97a3b~tplv-k3u1fbpfcp-zoom-1.image)

## 推送本地分支到远程仓库

#### 基本语法

**git push 别名 分支**



#### 案例实操 我记得这里需要登录账号

Layne@LAPTOP-Layne MINGW64 /d/Git-Space/SH0720 (master)

$ **git push ori master**

Logon failed, use ctrl+c to cancel basic credential prompt. 

Username for 'https://github.com': atguiguyueyue

Counting objects: 3, done.

Delta compression using up to 12 threads.

Compressing objects: 100% (2/2), done.

Writing objects: 100% (3/3), 276 bytes | 276.00 KiB/s, done. 

Total 3 (delta 0), reused 0 (delta 0)

To <https://github.com/atguiguyueyue/git-shTest.git>

* [new branch] master -> master

此时发现已将我们master 分支上的内容推送到GitHub 创建的远程仓库。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9b9bc643708b48ca83bcf740247f5495~tplv-k3u1fbpfcp-zoom-1.image)



## 克隆远程仓库到本地

#### 基本语法

**git clone 远程地址**

****

#### 案例实操

Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong

$ **git clone** [**https://github.com/atguiguyueyue/git-shTest.git**](https://github.com/atguiguyueyue/git-shTest.git)

Cloning into 'git-shTest'...

remote: Enumerating objects: 3, done. remote: Counting objects: 100% (3/3), done.

remote: Compressing objects: 100% (2/2), done.

remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0 Unpacking objects: 100% (3/3), done.

https://github.com/atguiguyueyue/git-shTest.git

这个地址为远程仓库地址，克隆结果：初始化本地仓库




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c1e60ac19ea842c0baee3471cead4adf~tplv-k3u1fbpfcp-zoom-1.image)

```
--创建远程仓库别名
Layne@LAPTOP-Layne MINGW64 /d/Git-Space/pro-linghuchong/git-shTest(master)
$ git remote -v
origin https://github.com/atguiguyueyue/git-shTest.git (fetch)
origin https://github.com/atguiguyueyue/git-shTest.git (push)
```

小结：clone 会做如下操作。1、拉取代码。2、初始化本地仓库。3、创建别名




## 邀请加入团队

#### 选择邀请合作者




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca240a136ac64ba2b35ece5e8f2ee73e~tplv-k3u1fbpfcp-zoom-1.image)




#### 填入想要合作的人




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca86f53a56204db5a319927f3bd03841~tplv-k3u1fbpfcp-zoom-1.image)




#### 复 制 地 址 并 通 过 微 信 钉 钉 等 方 式 发 送 给 该 用 户 ， 复 制 内 容 如 下 ：




[**https://github.com/atguiguyueyue/git-shTest/invitations**](https://github.com/atguiguyueyue/git-shTest/invitations)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ae49e9b1bfc4da3bbeba5a7ef344ccc~tplv-k3u1fbpfcp-zoom-1.image)




#### 在 atguigulinghuchong 这个账号中的浏览器地址栏复制收到邀请的链接，点击接受邀请。







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/28cc117723b14ac2a3dbdbcf44a3161c~tplv-k3u1fbpfcp-zoom-1.image)




**成功之后可以在 atguigulinghuchong 这个账号上看到 git-Test 的远程仓库。**




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71fa1d2668e849a68414d8271c86fc9c~tplv-k3u1fbpfcp-zoom-1.image)

####




## 拉取远程库内容

#### 14 基本语法

**git pull 远程库地址别名 远程分支名**




#### 15 案例实操

--将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并

Layne@LAPTOP-Layne MINGW64 /d/Git-Space/SH0720 (master)

$ **git pull ori master**

remote: Enumerating objects: 5, done. remote: Counting objects: 100% (5/5), done.

remote: Compressing objects: 100% (1/1), done.

remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0 Unpacking objects: 100% (3/3), done.

From <https://github.com/atguiguyueyue/git-shTest>

* branch master -> FETCH_HEAD 7cb4d02..5dabe6b master -> ori/master

Updating 7cb4d02..5dabe6b Fast-forward

hello.txt | 2 +-

1 file changed, 1 insertion(+), 1 deletion(-) Layne@LAPTOP-Layne MINGW64 /d/Git-Space/SH0720 (master)

## 跨团队协作

#### 将远程仓库的地址复制发给邀请跨团队协作的人，比如东方不败。










![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cfec4b8854804738bb686c24a08085a9~tplv-k3u1fbpfcp-zoom-1.image)




#### 在东方不败的 GitHub 账号里的地址栏复制收到的链接，然后点击 Fork 将项目叉到自己的本地仓库。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18d469f58b2747849f58a3c505f5b882~tplv-k3u1fbpfcp-zoom-1.image)







####




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84d4d9001b23404cb12ffc971901e0db~tplv-k3u1fbpfcp-zoom-1.image)




叉成功后可以看到当前仓库信息。










![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/645750bd300549bd954b7a6958fe3981~tplv-k3u1fbpfcp-zoom-1.image)




#### 东方不败就可以在线编辑叉取过来的文件。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e21a15418f974b24a80d92eb8889dedd~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/013bec53e86842fa8b789dc865e787ae~tplv-k3u1fbpfcp-zoom-1.image)






#### 编辑完毕后，填写描述信息并点击左下角绿色按钮提交。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97f91cc5221f4d77b4b53c2a4f57693b~tplv-k3u1fbpfcp-zoom-1.image)



#### 接下来点击上方的 Pull 请求，并创建一个新的请求。



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27645c55c79842dab05f251dc633e204~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/327db3f142be45ffb809862024b341c1~tplv-k3u1fbpfcp-zoom-1.image)



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f1d731b42fa4b5eb0ba8cea14f0a5da~tplv-k3u1fbpfcp-zoom-1.image)

#### 回到岳岳 GitHub 账号可以看到有一个 Pull request 请求。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f51239d523049adb0ec52d06dfefc58~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2319d76354c446c893b937fbc9f4e769~tplv-k3u1fbpfcp-zoom-1.image)


#### 进入到聊天室，可以讨论代码相关内容。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d722c9e2743412abcde16e4c213eda7~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1e8a861cef53453dada2461778970172~tplv-k3u1fbpfcp-zoom-1.image)




#### 如果代码没有问题，可以点击 Merge pull reque 合并代码。






![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05f8b77b133b4f67ab4100cf3c746843~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba39ef487b0e4ea3b133422f35c29773~tplv-k3u1fbpfcp-zoom-1.image)

