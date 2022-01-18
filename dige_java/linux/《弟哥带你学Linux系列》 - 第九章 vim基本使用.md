## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂
#

Vim有3种模式：一般模式、编辑模式和命令模式

## 一般模式

当我们使用命令vim filename编辑文件时，默认进入该文件的一般模式。在这个模式下，可以做的操作有：上下移动光标、删除某个字符、删除某行以及复制或粘贴一行或者多行。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed756e6461c447e79bc5ac4462761ac9~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b708abe1454d444694bbe9e1d7c6e0bd~tplv-k3u1fbpfcp-zoom-1.image)




## 编辑模式

在一般模式下不可以修改某一个字符，如果要修改字符只能进入编辑模式。从一般模式进入编

辑模式，只需按i 、l 、a 、A 、。、0 、r和R中的某一个键即可。当进入编辑模式时，在屏幕的尾行会显

示INSERT或REPLACE的字样（如果你的CentOS支持中文，则会显示“插入”）。从编辑模式回到一般

模式，只需按Esc键即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85ccb9e704db4f519c94d89d27cf0288~tplv-k3u1fbpfcp-zoom-1.image)



## 命令模式

在一般模式下，输入：或者／即可进入命令模式。在该模式下，我们可以搜索某个字符或者字符串，也可以实现保存、替换、退出、显示行号等操作，如表7-4所示。![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e082bf8943324a4aa55a7164dd9b9231~tplv-k3u1fbpfcp-zoom-1.image)

