---
公众号：弟哥带你进大厂
---
# 第 10 章 自建代码托管平台-GitLab



## GitLab 简介

GitLab 是由GitLabInc.开发，使用 [MIT 许可证](https://baike.baidu.com/item/MIT%E8%AE%B8%E5%8F%AF%E8%AF%81)的基于[网络](https://baike.baidu.com/item/%E7%BD%91%E7%BB%9C)的 [Git ](https://baike.baidu.com/item/Git)[仓库](https://baike.baidu.com/item/%E4%BB%93%E5%BA%93)管理工具，且具有[wiki ](https://baike.baidu.com/item/wiki)和 issue 跟踪功能。使用 [Git ](https://baike.baidu.com/item/Git)作为代码管理工具，并在此基础上搭建起来的web 服务。GitLab 由乌克兰程序员DmitriyZaporozhets 和 ValerySizov 开发，它使用 [Ruby 语言](https://baike.baidu.com/item/Ruby%E8%AF%AD%E8%A8%80)写成。后来，一些部分用 [Go 语言](https://baike.baidu.com/item/Go%E8%AF%AD%E8%A8%80)重写。截止 2018 年 5 月，该公司约有 290 名团队成员，以及 2000 多名开源贡献者。GitLab 被 IBM，Sony，JülichResearchCenter，NASA，Alibaba， Invincea，O’ReillyMedia，Leibniz-Rechenzentrum(LRZ)，CERN，SpaceX 等组织使用。

## GitLab 官网地址

官网地址：<https://about.gitlab.com/>




安装说明：<https://about.gitlab.com/installation/>



## GitLab 安装

### 服务器准备

准备一个系统为 CentOS7 以上版本的服务器，要求内存 4G，磁盘 50G。关闭防火墙，并且配置好主机名和 IP，保证服务器可以上网。

此教程使用虚拟机：主机名：gitlab-server IP 地址：192.168.6.200




### 安装包准备

Yum 在线安装 gitlab- ce 时，需要下载几百M 的安装文件，非常耗时，所以最好提前把所需RPM 包下载到本地，然后使用离线 rpm 的方式安装。

下载地址：

<https://packages.gitlab.com/gitlab/gitlab->ce/packages/el/7/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm

注：资料里提供了此rpm 包，直接将此包上传到服务器/opt/module 目录下即可。




### 编写安装脚本

安装 gitlab 步骤比较繁琐，因此我们可以参考官网编写 gitlab 的安装脚本。

[root@gitlab-server module]# vim gitlab-install.sh

sudo rpm -ivh /opt/module/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm

sudo yum install -y curl policycoreutils-python openssh-server cronie

给脚本增加执行权限

[root@gitlab-server module]# chmod +x gitlab-install.sh [root@gitlab-server module]# ll

总用量 403104

-rw-r--r--. 1 root root 412774002 4 月 7 15:47 gitlab-ce-13.10.2- ce.0.el7.x86_64.rpm

-rwxr-xr-x. 1 root root 416 4 月 7 15:49 gitlab-install.sh

sudo lokkit -s http -s ssh sudo yum install -y postfix sudo service postfix start sudo chkconfig postfix on

curl <https://packages.gitlab.com/install/repositories/gitlab/gitlab-> ce/script.rpm.sh | sudo bash

sudo [EXTERNAL_URL="http://gitlab.](http://gitlab.example.com/)example.com" yum -y install gitlab- ce

然后执行该脚本，开始安装 gitlab-ce。注意一定要保证服务器可以上网。

[root@gitlab-server module]# ./gitlab-install.sh

警告：/opt/module/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm: 头 V4 RSA/SHA1 Signature, 密钥 ID f27eab47: NOKEY

准备中... ################################# [100%]

正在升级/安装...

1:gitlab-ce-13.10.2-ce.0.el7 ################################# [100%]



## 初始化 GitLab 服务

执行以下命令初始化 GitLab 服务，过程大概需要几分钟，耐心等待…

[root@gitlab-server module]# gitlab-ctl reconfigure



Running handlers:

Running handlers complete

Chef Client finished, 425/608 resources updated in 03 minutes 08 seconds

gitlab Reconfigured!

## 启动 GitLab 服务

执行以下命令启动 GitLab 服务，如需停止，执行 gitlab-ctl stop

[root@gitlab-server module]# gitlab-ctl start 

ok: run: alertmanager: (pid 6812) 134s

ok: run: gitaly: (pid 6740) 135s

ok: run: gitlab-monitor: (pid 6765) 135s

ok: run: gitlab-workhorse: (pid 6722) 136s ok: run: logrotate: (pid 5994) 197s

ok: run: nginx: (pid 5930) 203s

ok: run: node-exporter: (pid 6234) 185s ok: run: postgres-exporter: (pid 6834) 133s ok: run: postgresql: (pid 5456) 257s

ok: run: prometheus: (pid 6777) 134s ok: run: redis: (pid 5327) 263s

ok: run: redis-exporter: (pid 6391) 173s ok: run: sidekiq: (pid 5797) 215s

ok: run: unicorn: (pid 5728) 221s




## 使用浏览器访问 GitLab

使用主机名或者IP 地址即可访问GitLab 服务。需要提前配一下 windows 的hosts 文件。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f070e520e27d497ba4f05e660327fc8e~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ef8ff2f595e49e3b374c2f4bdd7ac75~tplv-k3u1fbpfcp-zoom-1.image)










![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94a136aacfb442b5bde1aeba3b1a2b9c~tplv-k3u1fbpfcp-zoom-1.image)

首次登陆之前，需要修改下 GitLab 提供的 root 账户的密码，要求 8 位以上，包含大小写子母和特殊符号。因此我们修改密码为 Atguigu.123456

然后使用修改后的密码登录 GitLab。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2850d09d364f480fa5fc2ceb5acea744~tplv-k3u1fbpfcp-zoom-1.image)

GitLab 登录成功。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/662daedc76e9440bbfb224597395538d~tplv-k3u1fbpfcp-zoom-1.image)




## GitLab 创建远程库




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3638934268e7482ba570188dd04bb692~tplv-k3u1fbpfcp-zoom-1.image)![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/704b5416267a41a2b986e6a9a1ce4a85~tplv-k3u1fbpfcp-zoom-1.image)





![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80dd1db82b10470ba579e9b0795e7dcd~tplv-k3u1fbpfcp-zoom-1.image)

## IDEA 集成 GitLab

### 安装 GitLab 插件




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b17277d947a54297a9283017bcefc36d~tplv-k3u1fbpfcp-zoom-1.image)




### 设置GitLab 插件




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6bc58a2abea04cdeafa7839914189e75~tplv-k3u1fbpfcp-zoom-1.image)







![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fda9d18916fa432db89af94feec52127~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/198a24679ebe4a4f932d777d50349cfd~tplv-k3u1fbpfcp-zoom-1.image)




### push 本地代码到GitLab 远程库



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/66118ee442884895863c8eea3819ac17~tplv-k3u1fbpfcp-zoom-1.image)




自定义远程连接










![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14bd4fce4a7c46ff837cdb6039b94d0f~tplv-k3u1fbpfcp-zoom-1.image)




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a788f13f5f77457da8877bfc5949b315~tplv-k3u1fbpfcp-zoom-1.image)




注意：gitlab 网页上复制过来的连接是：<http://gitlab.example.com/root/git-test.git>， 需要手动修改为：<http://gitlab-server/root/git-test.git>

选择 gitlab 远程连接，进行 push。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/323231fcac6b40958f6356e6ffdd3721~tplv-k3u1fbpfcp-zoom-1.image)




首次向连接 gitlab，需要登录帐号和密码，用 root 帐号和我们修改的密码登录即可。




![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/48ba0c8154d74870b734a70aa30a8637~tplv-k3u1fbpfcp-zoom-1.image)

代码 Push 成功。


![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b97806e52d6444e08eedaaacd38e7316~tplv-k3u1fbpfcp-zoom-1.image)




只要 GitLab 的远程库连接定义好以后，对 GitLab 远程库进行 pull 和 clone 的操作和Github 和码云一致，此处不再赘述。

