---
公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂
---
首先在linux服务器上生成密钥

ssh-keygen -t rsa

然后设置密钥密码

之后私钥和公钥都会保存在/root/.ssh文件夹里

然后设置文件夹权限

chmod 700 /root/.ssh

由于ssh公钥需要保存在名为authorized_keys的文件里，故需要

cat .ssh/id_rsa.pub >> .ssh/authorized_keys

然后将密钥保存到本地

Disabled SElinux

vi /etc/seLinux/config

回车后,把光标移动到SELINUX=enforcing这一行,输人i进入编辑模式,修改为SELINUX= disabled。按Esc键，然后输入:wq并回车，完成设置

重启机器

reboot

之后就可以通过密钥连接服务器了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/74eebd1cbec64dbfb65b21d6987c67d2~tplv-k3u1fbpfcp-zoom-1.image)

用户密钥选择本地保存的私钥，并输入密钥密码，连接即可



