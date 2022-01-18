---
公众号：弟哥带你进大厂
---
# shell基础

## 记录命令历史 

我们执行过的命令Linux 都会记录，预设可以记录1000条历史命令 这些命令保存在用户的家目录的.bash_history文件中 但需要注意的是，只有当用户正常退出当前shell时，在当前shell中运行的命令才会保存至.bash_history文件中 

-   **!!：** 连续两个!表示执行上条指令，示例命令如下 

```
# pwd 
/root 
#!! 
pwd 
/root 
```

-     **!n：** 这里的n是数字，表示执行命令历史中的第n条指令 例如，!1002 表示执行命令历史中的第1002个命令，如下所示 ：

```
# history | grep 1002 
1002 pwd 
1015 history |grep 1002 
# !1002 
pwd 
/root
```

-   **!字符串（字符串大于等于1)** :例如!pw表示执行命令历史中最近一次以pw开头的命令。示例代码如下:

```
# !pw 
pwd 
/root
```

## 通配符 

在bash下，可以使用*来匹配零个或多个字符，用?匹配一个字符。示例命令如下:

```
[root@localhost dir3]# ls
11887022.html  file1.tar  g33  tesallt.zip  test  test2  test.bz2  test.zip  yasuo.tar
[root@localhost dir3]# ls -a ?est*
test  test2  test.bz2  test.zip
```

## 别名

也可以向定义命令的别名，其格式为 alias 命令别名 ＝ '具体的命令' ，示例命令如下： 

```
# alias aming = 'pwd' 
# aming 
/root 
# unalias aming 
# aming
bash: aming 未找到命令
```

## 输入／输出重定向 

输入重定向用于改变命令的输入，输出重定向用于改变命令的输出，输出重定向更为常用，它经常用于将命令的结果输入到文件中，而不是屏幕上。输入重定向的命令是<，输出重定向的命令是> 。另外，还有错误重定向命令2＞以及追加重定向命令>>

```
[root@localhost dir3]# echo "123123" >test
[root@localhost dir3]# echo "123123" >>test
[root@localhost dir3]# cat test
123123
123123

——————————————————————————————————

[root@localhost test]# ls aaa 2> error
[root@localhost test]# cat error
ls: 无法访问aaa: 没有那个文件或目录

[root@localhost test]# ls bbb 2>> error
[root@localhost test]# cat error
ls: 无法访问aaa: 没有那个文件或目录
ls: 无法访问bbb: 没有那个文件或目录
```

## 管道符 

前面已经提过管道符｜，它用于将前一个指令的输出作为后一个指令的输入，如下所示 

```
[root@localhost dir3]# cat /etc/passwd | wc -l
22
```

表示passwd有22行

## 作业控制 

当运行进程时，你可以使它暂停（按Ctrl+Z组合键)，然后使用fg ( foreground的简写)命令恢复它，或是利用

bg ( background的简写)命令使它到后台运行。此外，你也可以使它终止（按CtrI+C组合键）。示例命令如下:

```
[root@localhost dir3]# vi test //修改一些东西后，按ESC，CTRL+Z

[1]+  已停止               vi test
[root@localhost dir3]# jobs    //jobs 命令显示了当前 shell 环境中已启动的作业状态
[1]+  已停止               vi test
```

使用bg可以让作业在后台运行

```
[root@localhost dir3]# vmstat 1 > test2
^Z
[2]+  已停止               vmstat 1 > test2

[root@localhost dir3]# jobs     //多个被暂停的任务会有编号
[1]-  已停止               vi test
[2]+  已停止               vmstat 1 > test2


[root@localhost dir3]# bg 2 //让2号任务在后台运行
[2]+ vmstat 1 > test2 &
```

可以通过fg命令回到某个被暂停的任务

```
[root@localhost dir3]# fg 2
vmstat 1 > test2
```

结束任务有两种方法：

```
第一种：在某个任务执行页面按CTRL+C

[root@localhost dir3]# fg 2
vmstat 1 > test2
^C
[root@localhost dir3]# 

————————————————————————————————————————————————————————————————————————————

第二种：查找某任务的pid，使用 kill pid，若某个任务无法结束，可以用-9选项强行杀死

[root@localhost dir3]# ps -aux | grep vmstat
root       4135  0.0  0.0 152700  1408 pts/0    T    17:38   0:00 vmstat 1
root       4137  0.0  0.0 112824   976 pts/0    S+   17:39   0:00 grep --color=auto vmstat
[root@localhost dir3]# kill -9 4135
[root@localhost dir3]# jobs
[1]+  已杀死               vmstat 1 > test2
```

# 变量

在前面章节中介绍过环境变量PATH，它是shell预设的一个变量。**通常，shell预设的变量都是大写的**。变量就是使用一个较简单的字符串来替代某些具有特殊意义的设定以及数据。就拿PATH来讲,这个PATH就代替了所有常用命令的绝对路径的设定。有了PATH这个变量，我们运行某个命令时，就不再需要输入全局路径，直接输入命令名即可。你可以使用echo命令显示变量的值，如下所示:

```
[root@localhost dir3]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

## env

使用env命令、可列出系统预设的全部系统变量 

```
HOSTNAME:表示主机的名称。
SHELL:表示当前用户的shell类型。HISTSIZE:表示历史记录数。
MAIL:表示当前用户的邮件存放目录。
PATH:该变量决定了shell将到哪些目录中寻找命令或程序。PWD:表示当前目录。
LANG:这是与语言相关的环境变量，多语言环境可以修改此环境变量。HOME:表示当前用户的家目录。
LOGNAME:表示当前用户的登录名。
```

## set

env命令显示的变量只是环境变量，系统预设的变量其实还有很多，你可以使用set命令把系统预设的全部变量都显示出来。 set命令不仅可以显示系统预设的变量，也可以显示用户自定义的变量 比如，我们向定义一个变量 ，如下所示 ：

```
[root@localhost dir3]# myname=tiansuo
[root@localhost dir3]# echo $myname
tiansuo
[root@localhost dir3]# set | grep myname
myname=tiansuo
```

虽然你可以自定义变量，但是该变量只能在当前shell 生效 如果想让设置的环境变量一直生效，该怎么做呢?这分以下两种情况：

-   允许系统内所有用户登录后都能使用该变量：具体的操作方法是:在/etc/profile文件的最后一行加入export myname=tiansuo，然后运行source /etc/profile就可以生效了。此时再运行bash命令或者切换到其他账户（如su - test）就可以看到效果。如下所示:

```
[root@localhost dir3]# echo "export myname=tiansuo" >> /etc/profile
[root@localhost dir3]# source !$
source /etc/profile
[root@localhost dir3]# bash
[root@localhost dir3]# echo $myname
tiansuo
[root@localhost dir3]# su ts
[ts@localhost dir3]$ echo $myname
tiansuo
```

-   仅允许当前用户使用该变量：具体的操作方法是:在用户主目录下的.bashrc文件的最后一行加人export myname=Aming，然后运行source .bashrc就可以生效了。这时再登录test账户，myname变量则不会生效了。这里source命令的作用是将目前设定的配置刷新，即不用注销再登录也能生效。

使用unset $变量名 可以删除自定义的环境变量

## 在Linux下设置向定义变量的规则

-   设定变量的格式为a=b，其中a为变量名,b为变量的内容，等号两边不能有空格。
-   变量名只能由字母、数字以及下划线组成,而且不能以数字开头。

<!---->

-   当变量内容带有特殊字符（如空格)时，需要加上单引号。示例命令如下:

```
		[root@localhost dir3]# myname='tian suo hao er'
		[root@localhost dir3]# echo $myname
		tian suo hao er
```

-   若是变量内容中本身带有单引号，这时就需要加双引号了，示例命令如下 

```
[root@localhost dir3]# myname="tiansuo''''haoer"
[root@localhost dir3]# echo $myname
tiansuo''''haoer
```




-   如果变量内容中需要用到其他命令的运行结果则可以使用反引号，示例命令如下： 

```
[root@localhost dir3]# myname=tiansuo`pwd`
[root@localhost dir3]# echo $myname
tiansuo/root/dir3
```

-   变量内容可以累加其他变量的内容，但需要加双引号，示例命令如下： 

```
[root@localhost dir3]# hername=wobushi"$myname"
[root@localhost dir3]# echo $hername
wobushitiansuo/root/dir3
```

在父 hell 中设定变量后，进入子shell 时，该变量是不会生效的 如果想让这个变量在子shell生效，则要用到export 指令 

```
# abc=123 
# echo $abc 
123 
# bash 
# echo $abc 
# exit 
exit 
# export abc 
# echo $abc 
123 
# bash 
# echo $abc 
123
```

# 系统环境变量与个人环境变量的配置文件 

-   **/etc/profile**:这个文件预设了几个重要的变量，例如PATH、USER、LOGNAME、MAIL、INPUTRC、HOSTNAME、HISTSIZE、umask等。
-   **/etc/bashrc**:这个文件主要预设umask以及PS1。这个PS1就是我们在输入命令时前面的那串字符。例如，我的Linux系统的PS1就是[root@localhost~]#，我们不妨看一下PS1的值，如下所示:

```
[root@localhost dir3]# echo $PS1
[\u@\h \W]$
```

其中，\u指用户，\h指主机名,\W指当前目录,\$指字符#（如果是普通用户，则显示为$)。

除了以上两个系统级别的配置文件外，每个用户的主目录下还有以下几个隐藏文件。

-   **.bash_profile**:该文件定义了用户的个人化路径与环境变量的文件名称。每个用户都可使用该文件输入专属于自己的shell信息，当用户登录时,该文件仅仅执行一次。
-   **.bashrc**:该文件包含专属于自己的shell的bash信息，当登录或每次打开新的shell时，该文件会被读取。例如，你可以将用户自定义的别名或者自定义变量写到这个文件

<!---->

-   **.bash_history**:该文件用于记录命令历史。
-   .**bash_logout**:当退出shell时，会执行该文件。你可以将一些清理的工作放到这个文件中。

# Linux shell 中的特殊符号 

## [通配符]()

## 注释符号＃ 

这个符号在linux 中表示注释说明，即＃后面的内容都会被忽略 用法如下：

```
 # abc=123 #aaaaa 
 # echo $abc 
 123  
```

## 脱义字符＼ 

这个字符会将后面的特殊符号（如*)还原为普通字符。用法如下:

```
#ls -d test*
ls:无法访问test*:没有那个文件或目录
```





## 特殊符号 ； && ||

通常，我们都是在一行中输入一个命令，然后回车就运行了。如果想在一行中运行两个或两个以上的命令，需要在命令之间加符号;。示例命令如下:

```
[root@localhost test]# mkdir testdir ; touch test1.txt ; touch test2.txt; ls
test1.txt  test2.txt  testdir
```

使用;时，不管command1是否执行成功，都会执行command2。

使用8&时，只有command1执行成功后，command2才会执行，否则command2不执行。

使用||时，command1执行成功后则command2不执行，否则执行command2，即command1和command2中总有一条命令会执行。

## 特殊符号 ~

符号~表示用户的家目录,root用户的家目录是/root，普通用户则是/home/username。示例命令如下:

```
[root@localhost test]# cd ~
[root@localhost ~]# ls
anaconda-ks.cfg  test
```

## 特殊符号＆ 

如果想把一条命令放到后台执行，则需要加上符号&，它通常用于命令运行时间较长的情况。比如，可以用在sleep后，如下所示:

```
[root@localhost ~]# sleep 30 &
[1] 4585
[root@localhost ~]# jobs
[1]+  运行中               sleep 30 &
```

## 中括号[]

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f4aa9cb57de4828a503d1f1444efcfe~tplv-k3u1fbpfcp-zoom-1.image)

