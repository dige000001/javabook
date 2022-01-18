## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 查看磁盘或者目录的容量

## 命令df

命令df ( disk filesystem的简写）用于查看已挂载磁盘的总容量、使用容量、剩余容量等，可以不

加任何参数，默认以KB为单位显示。示例命令如下：

```
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 898M     0  898M    0% /dev
tmpfs                    910M     0  910M    0% /dev/shm
tmpfs                    910M  9.6M  901M    2% /run
tmpfs                    910M     0  910M    0% /sys/fs/cgroup
/dev/mapper/centos-root   48G  1.4G   47G    3% /
/dev/mapper/centos-home   47G   33M   47G    1% /home
/dev/sda1               1014M  151M  864M   15% /boot
tmpfs                    182M     0  182M    0% /run/user/0
```

上例的结果中，其中/、/boot是我们在安装系统时划分出来的。/dev 、/dev/shm为内存分区，默认大

小为内存大小的1/2 ，如果我们把文件存到这个分区下，相当于存到了内存中，好处是读写非常快，坏处是系统重启时文件就会丢失。后面的/run 、/sys/fs/cgroup等分区都是tmpfs ，跟／dev/shm类似，为临时

文件系统，我们不要碰它们。df命令的常用选项有-i 、-h 、-k和-m ，下面介绍这4个选项的用法。

-   **-i ：** 表示查看inodes 的使用状况，如使已用100% ，即使磁盘空间有富余，也会提示磁盘空间已满。示例命令如下：

```
[root@localhost ~]# df -ih | grep -v tmpfs
文件系统                Inode 已用(I) 可用(I) 已用(I)% 挂载点
/dev/mapper/centos-root   24M     28K     24M       1% /
/dev/mapper/centos-home   24M      14     24M       1% /home
/dev/sda1                512K     327    512K       1% /boot
```

-   **-h** ：表示使用合适的单位显示，
-   -k , -m ：分别表示以KB和MB为单位显示。

## du

[命令du ( disk useage ）用来查看某个目录或文件所占空间的大小，其格式为du [ -abckmsh] ［文件 或者目录名］](https://www.yuque.com/normalgamer/msmcvb/kh1eoh#iXUbc)

习惯用du -sh 干ilename这样的形式

# 磁盘分区和格式化

oCrtc:结束(终止)当前命令.如果你输人了一大审字特,但不想运行,可以按CH+C组合

键,此时光标将跳人下一行,而在刚刚的光标处会留下一个)C的标记,如图2-44所示.

mlaksdirasjacklasfac

[rootelocalhosL

Lrootelocalhosl

1社

取消命令

图2-44

Tab:实现自动补全功能.这个键比较重要,使用频率也很高.当你输人命令,文件或目录

的前凡个字符时,它会自动帮你补全.比如,前面阿铭教大家编锌网卡的配置文件时文件路

径很长,这时结合Tab键就会很轻松.

口Ctrl+D:退出当前终端.同样,你也可以输入命令exit实现该功能.

Cmtz:暂停当前进程.这和CHlC是有去区别的,暂停后,使用命令恢复该进程,该知

识点会在第10章中介绍到.

Cm+L:清屏,使光标移动到屏幕的第一行.当命令和显示的结果占满整屏幕时,我们每

运行一个命令,都会在最后一行显示,这样看起来不太方便,此时就可以使用这个快捷键,

让光标移动到屏幕第一行,也就是所谓的清屏.

CHitA:可以让光标移动到命令的最前面.有时候一条命令很长,快完时发现前面个

字母不对,此时可以直接用这个快捷键把光标定位到行首,然后再用左右方向键微调光标的

位置.

Ctrl+E:可以让光标移动到最后面,作用同上.

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb0f4870945244e9b668a462e03a0ab1~tplv-k3u1fbpfcp-zoom-1.image)

