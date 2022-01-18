# 第1章 Git 概述


Git 是一个免费的、开源的分布式版本控制系统，可以快速高效地处理从小型到大型的各种项目。

Git 易于学习，占地面积小，性能极快。 它具有廉价的本地库，方便的暂存区域和多个工作流分支等特性。其性能优于 Subversion、CVS、Perforce 和 ClearCase 等版本控制工具。

## 1 何为版本控制

版本控制是一种记录文件内容变化，以便将来查阅特定版本修订情况的系统。

版本控制其实最重要的是可以记录文件修改历史记录，从而让用户能够查看历史版本， 方便版本切换。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8401c66e05ab4c6b848b93cdede2b2fe~tplv-k3u1fbpfcp-zoom-1.image)



## 2 为什么需要版本控制

个人开发过渡到团队协作。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d33e2f619724926a5b88f1252db0284~tplv-k3u1fbpfcp-zoom-1.image)




## 3 版本控制工具

#### 集中式版本控制工具




CVS、SVN(Subversion)、VSS……




集中化的版本控制系统诸如 CVS、SVN 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。多年以来，这已成为版本控制系统的标准做法。

这种做法带来了许多好处，每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统，要远比在各个客户端上维护本地数据库来得轻松容易。

事分两面，有好有坏。这么做显而易见的缺点是中央服务器的单点故障。如果服务器宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c14ef1b1497a40ff96644b5cf73e8275~tplv-k3u1fbpfcp-zoom-1.image)

#### 分布式版本控制工具

Git、Mercurial、Bazaar、Darcs……



像 Git 这种分布式版本控制工具，客户端提取的不是最新版本的文件快照，而是把代码仓库完整地镜像下来（本地库）。这样任何一处协同工作用的文件发生故障，事后都可以用其他客户端的本地仓库进行恢复。因为每个客户端的每一次文件提取操作，实际上都是一次对整个文件仓库的完整备份。

分布式的版本控制系统出现之后,解决了集中式版本控制系统的缺陷:




1.  服务器断网的情况下也可以进行开发（因为版本控制是在本地进行的）
1.  每个客户端保存的也都是整个完整的项目（包含历史记录，更加安全）










![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a2ced4252474142bc42e0c5be221629~tplv-k3u1fbpfcp-zoom-1.image)




## 4 Git 简史

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e88df9546554ac1b4b6f88897d237b7~tplv-k3u1fbpfcp-zoom-1.image)



## 5 Git 工作机制

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69e006d78cec4adfa95b20846072296a~tplv-k3u1fbpfcp-zoom-1.image)

## 6 Git 和代码托管中心

代码托管中心是基于网络服务器的远程代码仓库，一般我们简单称为远程库。




### 局域网

-   -   -   -   GitLab

### 互联网

-   -   -   -   GitHub（外网）
            -   Gitee 码云（国内网站）