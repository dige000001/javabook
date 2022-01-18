## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

## man

man命令用于查看命令的帮助文档,其格式为 man 命令，如

man ls

## tail

tail [参数] [文件]

从尾部查看文本，默认10 行 -n 显示n行

tail -f 查看动态文本，即会动态更新最后十行并显示

## head

tail [参数] [文件]

从头查看文本，默认10 行 -n 显示n行

## grep

grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 **-**，则 grep 指令会从标准输入设备读取数据。

grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]

#

## shutdown 关机指令

shutdown [OPTIONS] [TIME] [WALL]

**OPTIONS：**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1f9c36890bf4756877ce47360b8ca76~tplv-k3u1fbpfcp-zoom-1.image)

**TIME**

两种方式：

1.  hh:mm，24小时制，表示在几点几分关机
1.  +m，m表示过几分钟后关机 now 相当于+0 

默认为+1

wall是一个信息，发给所有现在登录这台机器的人，告诉他们要关机了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69747a172c584764bcbaadb8cd659c52~tplv-k3u1fbpfcp-zoom-1.image)

关机的时候 输入 shutdown -c可以取消关机

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc52239d2a264b009abe3db8568222e6~tplv-k3u1fbpfcp-zoom-1.image)

\


## rm

用于删除文件

rm -r

可用于删除目录

rm -f

表示强制删除。它不再询问是否删除，而是直接删除。如果后面跟一个不存在的文件或者目录，则不会报错。慎用

## 环境变量PATH

在讲环境变量之前,先介绍一下命令**which**，它用于查找某个命令的绝对路径。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4a883e455e14db8b54b44be14f88117~tplv-k3u1fbpfcp-zoom-1.image)

表示我们用rm时，相当于用rm -i

但直接用/bin/rm时，是相当于不加任何参数的rm

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7249b423952489888c2244cacffd7bd~tplv-k3u1fbpfcp-zoom-1.image)

这便是环境变量了，干啥用的不用多说

### 新增环境变量：

export PATH=$PATH:/xxx

用于将xxx目录加入环境变量，需要注意： 直接使用 export 设置的变量都是**临时变量**，也就是说退出当前的 shell ，为该变量定义的值便不会生效了

#### 永久新增环境变量

**法一： 修改 /etc/profile**

vi /etc/profile

export PATH=$PATH:/xxx # 在配置文件中加入此行配置

退出后

source /etc/profile #修改完这个文件必须要使用这个命令在，使不用重启系统的情况下修改的内容生效。

**法二：修改 .bashrc 文件是在当前用户 shell 下生效**

vi /root/.bashrc?

在里面加入：

export PATH=$PATH:/xxx

修改这个文件之后同样也需要使用 source 或者是 . 使配置文件生效。

## touch

用于创建文件

## cp

cp是copy(即复制）的简写，该命令的格式为: cp [选项] [来源文件][目的文件]。下面介绍命令cp的几个常用选项。

cp -r

用于复制目录

## mv

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/774f740e9c6d4a8686a861eb68c49ad4~tplv-k3u1fbpfcp-zoom-1.image)

注意：第一第三种情况如何分辨：如果源文件是目录，则是第一种情况，源文件是文件，则是第三种情况

## cat

命令cat(它并不是某个单词的简写，大家可以通过man cat命令查看它的解释）是比较常用的一个命令，**用于查看一个文件的内容并将其显示在屏幕上**。cat后面可以不加任何选项，直接跟文件名。下面介绍它的两个常用选项：

-n 查看文件时，把行号也显示到屏幕上。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc3e2d6205ea470fada81ca1df1e97f9~tplv-k3u1fbpfcp-zoom-1.image)

上例中出现了符号>>，它跟前面介绍的符号>类似，其作用也是重定向，即把前面的内容输入到后面的文件中，但**符号>>是“追加”的意思。当使用符号>时，如果文件中有内容，则会删除文件中原有的内容，而使用符号>>则不会删除原有的内容。**

-A 显示所有的内容，包括特殊字符

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/241508dc412143df90896ea68f610eda~tplv-k3u1fbpfcp-zoom-1.image)

上例中，若不加-A选项,那么每行后面的$符号是看不到的。

## tac

反向cat，倒序显示

## more

命令more也用于查看一个文件的内容，后面直接跟文件名。当文件内容太多，一屏不能全部显示时，用命令cat肯定是看不了前面的内容，这时可以使用命令more。当看完一屏后，按空格键可以继续看下一屏，看完所有内容后就会退出，按Ctrl+D可以向上翻屏，按Ctrl+F向下翻屏（同空格)。如果你想提前退出,按q键即可。

## less

命令less的作用和命令more一样，后面直接跟文件名，但命令less比more功能要多一些。按空格键可以翻页，按j键可以向下移动(按一下就向下移动一行)，按k键可以向上移动。在使用more和les:查看某个文件时，你可以按一下/键，并输入一个字符串(（如root)，然后回车，这样就可以查找这个字符串了。如果是查找多个该字符串，可以按n键显示下一个。另外，也可以用?键替代/键来搜索字符串，唯一不同的是，/是在当前行向下搜索,而?是在当前行向上搜索。

## pwd

执行 pwd 指令可立刻得知您目前所在的工作目录的绝对路径名称。

## du

du命令用来计算文件或者目录的大小，-k表示以KB为单位

```
[root@localhost ~]# du -k
12	./.ssh
0	./dir3/g
32	./dir3
76	.

[root@localhost dir3]# du -k test
4	test
```

\


## cut

cut命令用来截取某一个字段，其格式为**cut -d '分隔字符' [-cf] n**，这里的n是数字。该命令有如下几个可用选项。

```
-d:后面跟分隔字符,分隔字符要用单引号括起来。
-c:后面接的是第几个字符。
-f:后面接的是第几个区块。即被分隔字符分隔的第几个
```

实例：

```
# head -5 /etc/passwd | cut -d ':' -f 1
root
bin
daemon
adm
lp

————————————————————————————————————————
[root@localhost dir3]# head -5 /etc/passwd | cut -d ':' -f 2
x
x
x
x
x

————————————————————————————————————————
[root@localhost dir3]# head -5 /etc/passwd | cut -d ':' -f 1-3
root:x:0
bin:x:1
daemon:x:2
adm:x:3
lp:x:4
```

```
[root@localhost dir3]# head -5 /etc/passwd | cut -c 1
r
b
d
a
l
```

## sort 

sort命令用做排序，其格式为**sort [-t分隔符][-kn1,n2] [-nru]** ，这里n1和n2指的是数字，其他选项的含义如下。

```
-t:后面跟分隔字符，作用跟cut的-d选项一样。
-n:表示使用纯数字排序。
-r:表示反向排序。
-u:表示去重复。
-kn1,n2:表示由n1区间排序到n2区间，可以只写-kn1，即对第n1个字段排序。
```

如果sort不加任何选项，则从首字符向后依次按ASCII码值进行比较，最后将它们按升序输出。

```
[root@localhost dir3]# head -5 /etc/passwd | sort
adm:x:3:4:adm:/var/adm:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
root:x:0:0:root:/root:/bin/bash
```

-t选项后面跟分隔符，-k选项后面跟单个数字表示对第几个区域的字符串排序，-n选项则表示使用纯数字排序。示例命令如下:

```
[root@localhost dir3]# head -5 /etc/passwd | sort -t: -k3 -n
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
```

-k选项后面跟数字n1和n2表示对第n1和n2区域内的字符串排序，-r选项则表示反向排序。示例命令如下:

```
[root@localhost dir3]# head -5 /etc/passwd | sort -t: -k3,4 -r
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
root:x:0:0:root:/root:/bin/bash
```

## wc

wc命令用于统计文档的行数、字符数或词数。该命令的常用选项有-1(统计行数)、-m(统计字符数）和-w(统计词数)。示例命令如下:

```
[root@localhost dir3]# cat /etc/passwd | wc -l
22


[root@localhost dir3]# cat /etc/passwd | grep adm | wc -l
1
```

\


## uniq 

uniq命令用来删除重复的行，该命令只有-c选项比较常用，它表示统计重复的行数，并把行数写在前面。我们先来编写一个文件,示例命令如下:

```
[root@localhost dir3]# cat test2
11111
11111
2222222
33333333
444
444
11111
444
——————————————————————————————————————————————————————————————
[root@localhost dir3]# uniq test2
11111
2222222
33333333
444
11111
444
```

可以看到，若不排序，只会去除相邻的重复行

```
[root@localhost dir3]# sort test2 | uniq
11111
2222222
33333333
444
[root@localhost dir3]# sort test2 | uniq -c
      3 11111
      1 2222222
      1 33333333
      3 444
```

注：uniq并不会真的删除原文件里的行，而是将去重的结果输出出来

```
[root@localhost dir3]# cat test2 //内容不变
11111
11111
2222222
33333333
444
444
11111
444
```

## tee 

tee命令后面跟文件名，其作用类似于重定向>，但它比重定向多一个功能，即把文件写入后面所跟的文件时，还显示在屏幕上。该命令常用于管道符|后。示例命令如下:

```
#echo "aaaaaaaaaaaaaaaaaaaaaaaaaaa"|tee testb.txt
aaaaaaaaaaaaaaaaaaaaaaaaaaa
#cat testb.txt
aaaaaaaaaaaaaaaaaaaaaaaaaaa
```

## tr

tr命令用于替换字符，常用来处理文档中出现的特殊符号，如DOS文档中出现的符号^W。该命令常用的选项有以下两个。

```
-d:表示删除某个字符,后面跟要删除的字符。
-s:表示删除重复的字符。
```

tr命令常用于把小写字母变成大写字母，如tr '[a-z]' '[A-Z]'。示例命令如下:

```
[root@localhost dir3]# head -2 /etc/passwd | tr '[a-z]' '[A-Z]'
ROOT:X:0:0:ROOT:/ROOT:/BIN/BASH
BIN:X:1:1:BIN:/BIN:/SBIN/NOLOGIN

[root@localhost dir3]# head -2 /etc/passwd | tr 'r' 'R'
Root:x:0:0:Root:/Root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

同样并不是真的对文件进行修改

## split 

split命令用于切割文档，**会真的修改文件**，常用的选项为-b和-l。

-   **-b:表示依据大小来分割文档,单位为byte**，示例命令如下:

```
[root@localhost g33]# cp /etc/passwd ./
[root@localhost g33]# split -b 500 passwd
[root@localhost g33]# ls
passwd  xaa  xab
```

如果split不指定目标文件名．则会以xaa xab...这样的文件名来存取切割后的文件 当然，我们也可以指定目标文件名，如下所示： 

```
[root@localhost g33]# split -b 500 passwd g!
[root@localhost g33]# ls
g!aa  g!ab  passwd
```

-   **-l:表示依据行数来分割文档**，示例命令如下:

```
[root@localhost g33]# split -l 10 passwd
[root@localhost g33]# ls
passwd  xaa  xab  xac
```

## date

date命令在shell脚本中最常用的几个用法如下。

```
date +%Y:表示以四位数字格式打印年份。
date +%y:表示以两位数字格式打印年份。
date +%m:表示月份。
date +%d:表示日期。
date +%H:表示小时。
date +%M:表示分钟。
date +%S:表示秒。
date +%w:表示星期。结果显示0则表示周日。
```

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

## 通配符 * ?

在bash下，可以使用*来匹配零个或多个字符，用?匹配一个字符。示例命令如下:

```
[root@localhost dir3]# ls
11887022.html  file1.tar  g33  tesallt.zip  test  test2  test.bz2  test.zip  yasuo.tar
[root@localhost dir3]# ls -a ?est*
test  test2  test.bz2  test.zip
```

## 别名 alias

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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b320ce19bd341fd8b84112ce6f8c1c3~tplv-k3u1fbpfcp-zoom-1.image)

