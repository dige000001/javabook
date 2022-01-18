---
公众号：弟哥带你进大厂
---
# grep/egrep 工具的使用 

注：建议查找的字符或字符串都加上''

## grep

该命令的格式为： **grep [-cinvABC] 'word' filename**

```
-c:表示打印符合要求的行数。
-i:表示忽略大小写。
-n:表示输出符合要求的行及其行号。
-v:表示打印不符合要求的行。
-A:后面跟一个数字（有无空格都可以)，例如-A2表示打印符合要求的行以及下面两行。
-B:后面跟一个数字，例如-B2表示打印符合要求的行以及上面两行。
-C:后面跟一个数字，例如-C2表示打印符合要求的行以及上下各两行。
```

**^后面跟的字符表示以该字符开头的一行**

```
[root@localhost test]# grep -v '#' error
ls: 无法访问aaa: 没有那个文件或目录
ls: 无法访问bbb: 没有那个文件或目录

[root@localhost test]# grep -v '^#' error  //不显示以#开头的一行
ls: 无法访问aaa: 没有那个文件或目录
ls: 无法访问bbb: 没有那个文件或目录
123#44
as#d11111
[root@localhost test]# grep -v '^^#' error  //不显示以^#开头的一行
```

**$前面跟的字符表示以该字符为结尾的一行，所以空行可以用^$表示**

```
[root@localhost test]# grep '$' error //如果要查找含有$的行，要用脱义字符\
$sssssssss
^$sssssss
ssss^$

[root@localhost test]# grep ^$ error //查找空行

[root@localhost test]# grep ^'$' error //查找以$开头的行
$sssssssss
```

几个技巧：

-   **不打印不以英文字母开头的行**

```
[root@localhost test]# grep -v ^[^a-zA-Z] error
ls: 无法访问aaa: 没有那个文件或目录
ls: 无法访问bbb: 没有那个文件或目录
ssss13444
ssss1ss3
```




-   **不打印纯英文的行——打印不全是英文的行。**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/403f17df279142d2a0eac81c1108202f~tplv-k3u1fbpfcp-zoom-1.image)

**由上可以看出： [^字符]表示除[]内字符之外的字符**



-   **过滤出任意一个字符和重复字符**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c03c90517788488e9789bac8b86abcd1~tplv-k3u1fbpfcp-zoom-1.image)

**.** 表示任意一个字符。上例中，r.o表示把r与o之间有一个任意字符的行过滤出来。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3572ed2cd814129b390c6b352e8e67f~tplv-k3u1fbpfcp-zoom-1.image)

*表示零个或多个*前面的字符。ooo*表示oo、ooo、oooo...或者更多的o。

-   **指定要过滤出的字符出现次数**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce83cc70cfdf43dbb500fdc59be376e3~tplv-k3u1fbpfcp-zoom-1.image)

这里用到了符号{}，其内部为数字，表示前面的字符要重复的次数。需要强调的是，{}左右都需要加上

转义字符\。另外，使用“{}”还可以表示一个范围，具体格式为{n1,n2}，其中n1< n2，表示重复n1到n2次前面的字符,n2还可以为空，这时表示大于等于n1次。

## egrep

-    **过滤出一个或多个指定的字符**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7b46a459bc84715bc3943d7d8f48913~tplv-k3u1fbpfcp-zoom-1.image)

和grep不同，这里egrep使用的是符号+，它表示匹配1个或多个+前面的字符，这个“+”是不支持被grep直接使用的。包括上面的，也是可以直接被egrep使用，而不用加\转义。示例如下:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a2f59b03bc14661a0c84597ea4d6a88~tplv-k3u1fbpfcp-zoom-1.image)

-    **过滤出零个或一个指定的字符**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79d0bf93adae467a8ef12d85a624bd98~tplv-k3u1fbpfcp-zoom-1.image)

*和?一样，表示有前面字符的0个或多个

-   **过滤出字符串1或者字符串2**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97af5bc48b68419395131ae9ff210b84~tplv-k3u1fbpfcp-zoom-1.image)

-   **egrep 中()的应用**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a99da88b292f414cb2e5bba6279e8da7~tplv-k3u1fbpfcp-zoom-1.image)

这里用()表示一个整体，上例中会把包含rooo或者rato的行过滤出来，另外也可以把()和其他符号组合在一起，例如(oo)+就表示1个或者多个oo。如下所示:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d10e9fabb06c4412913d8008f1a3517c~tplv-k3u1fbpfcp-zoom-1.image)



# sed

sed工具以及后面要介绍的awk工具就能把替换的文本输出到屏幕上，而且还有其他更丰富的功能。sed和awk都是流式编辑器，是针对文档的行来操作的。

### 打印某行

sed命令的格式为: **sed -n 'n'p filename**，单引号内的n是一个数字，表示第几行。-n选项的作用是只显示我们要打印的行，无关紧要的内容不显示。示例命令如下:

```
[root@localhost test]# sed -n '2'p passwd
bin:x:1:1:bin:/bin:/sbin/nologin
——————————————————————————————————
[root@localhost test]# sed -n '1,$'p passwd //打印所有行
[root@localhost test]# sed -n '1,3'p passwd //打印第一到第三行
```

### 打印包含某个字符串的行 

```
[root@localhost test]# sed -n '/root/'p passwd //单引号里面要查找字符的两边要加上/
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```

这种用法就类似于grep了，在grep中使用的特殊字符(如^、$、.、*等）同样也能在sed中使用

也可以这样：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f503b6a131e04f7bbebf433e257aa145~tplv-k3u1fbpfcp-zoom-1.image)

sed命令加上-e选项可以实现多个行为，如下所示 ：

```
[root@localhost test]# sed -ne '/root/'p -ne '2'p passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
```

### 删除某些行 

```
[root@localhost test]# sed '1'd passwd  //不显示第一行
bin:x:1:1:bin:/bin:/sbin/nologin
...
```

这里参数d表示删除的动作，它不仅可以删除指定的单行以及多行，而且可以删除匹配某个字符的行，还可以删除从某一行开始到文档最后一行的所有行。不过，这个操作仅仅是在显示器屏幕上并不显示这些行而已，文档还好好的,请不要担心。

### 替换字符或者字符串 

```
[root@localhost test]# sed  '1,2s/ot/to/g' passwd | head -n2
roto:x:0:0:roto:/roto:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

上例中的参数s就表示替换的动作，参数g表示本行全局替换，如果不加g则只替换本行出现的第一个，这个用法其实和vim的替换大同小异。

除了可以使用/作为分隔符外，我们还可以使用其他特殊字符，例如#和@

这个命令，**加上-i就会直接对文件进行操作**

****

# awk

### 截取文档中的某个段 

```
[root@localhost test]# head -n2 passwd | awk -F ':' '{print $1}'
root
bin
```

本例中，-F选项的作用是指定分隔符。如果不加-F选项，则以空格或者tab为分隔符。print为打印的动作，用来打印某个字段。$1为第1个字段，$2为第2个字段，以此类推。但$0比较特殊,它表示整行

注意awk的格式，-F后面紧跟单引号，单引号里面为分隔符。print的动作要用{}括起来，否则会报错。print还可以打印自定义的内容，但是自定义的内容要用双引号括起来，如下所示:

```
[root@localhost test]# head -n2 passwd | awk -F ':' '{print $1"###"$2"!!!"}'
root###x!!!
bin###x!!!
```

### 让某个段匹配字符或者字符串 

```
[root@localhost test]# awk -F ':' '$1 ~/oo/' passwd 
root:x:0:0:root:/root:/bin/bash
```

它可以让某个段去匹配,这里的~就是匹配的意思。

还可以多次匹配：

```
[root@localhost test]# awk -F ':' '/root/ {print $1,$5} /bash/ {print $7}'  passwd 
root root
/bin/bash
operator operator
/bin/bash
/bin/bash
/bin/bash
```

这里，先匹配满足有root字符串的行，并输出第一和第五个段，再匹配有bash字符串的行，并输出第七个段

### 条件操作符 

```
[root@localhost test]# awk -F ':' '$1=="root"' passwd 
root:x:0:0:root:/root:/bin/bash
```

上例为匹配第一个字段为root的行；

awk中可以用逻辑符号进行判断，比如==就是等于，也可以理解为精确匹配。另外还有>、>=、<=、!=、||、&&等。值得注意的是，在和数字比较时，若把比较的数字用双引号引起来，那么awk不会认为是数字，而会认为是字符,不加双引号则会认为是数字。

### awk 的内重变量 

awk常用的变量有OFS、NF和NR。**OFS和-F选项有类似的功能，也是用来定义分隔符的**，但是它是在输出的时候定义，**NF表示用分隔符分隔后一共有多少段，NR表示行号**。

#### OFS和NF

```
[root@localhost test]# head -5 passwd | awk -F ':' '{OFS="###"} {print $1,$NF}' //$NF表示最后一段
root###/bin/bash
bin###/sbin/nologin
daemon###/sbin/nologin
adm###/sbin/nologin
lp###/sbin/nologin

————————————————————————————————————————————
//高级用法
[root@localhost test]# head -5 passwd | awk -F ':' '{OFS="###"} {if($3>0) {print $1,$3}}'
bin###1
daemon###2
adm###3
lp###4
```

**OFS使用时要加{}**

#### NR

```
[root@localhost test]# awk -F ':' 'NR<3' passwd 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

**NR不加{}**

****

### awk 中的数学运算 

awk可以更改段值，示例命令如下 

```
[root@localhost test]# head -5 passwd | awk -F ':' '$1="wu"'
wu x 0 0 root /root /bin/bash
wu x 1 1 bin /bin /sbin/nologin
wu x 2 2 daemon /sbin /sbin/nologin
wu x 3 4 adm /var/adm /sbin/nologin
wu x 4 7 lp /var/spool/lpd /sbin/nologin
```

awk也可以对各个段的值进行数学运算，示例命令如下： 

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26ee48bc24444693a6a1a22d28ae2760~tplv-k3u1fbpfcp-zoom-1.image)


