---
公众号：弟哥带你进大厂
---
**建议凡是自定义的脚本都放到/usr/local/sbin/目录**

这样做的目的是：一来可以更好地管理文档；二来以后接管你工作的管理员都知道自定义脚本放在哪里，方便维护 

# shell 脚本的创建和执行 

```
[root@localhost sbin]# vim first.sh
输入:
#! /bin/bash 
date
fffffff
[root@localhost sbin]# sh first.sh 
2021年 09月 08日 星期三 09:53:51 CST
fffffff
```

#! /bin/bash 表明该脚本以bash语法写成

还有一种方法可以执行脚本

```
[root@localhost sbin]# chmod +x first.sh 
[root@localhost sbin]# ./first.sh 
2021年 09月 08日 星期三 09:55:28 CST
fffffff
```

这种方法需要文件有x权限

# shell脚本中的变量

定义变量的格式为:“变量名=变量的值”。在脚本中引用变量时需要加上符号$，这跟前面介绍的在shell中自定义变量是一致的。

```
修改first.sh：

#!/bin/bash
d=`date`
echo "the time is : $d"
echo "the time is : $d"
d1="sssss"
echo $d1

[root@localhost sbin]# ./first.sh 
the time is : $d
the time is : 2021年 09月 08日 星期三 10:03:25 CST
sssss
```

本例中使用到了**反引号,它的作用是将引号中的字符串当成shell命令执行,返回命令的执行结果。**

## 数学运算

```
#!/bin/bash
#the name of this file is "first.sh"

a=1
b=2
c=$a+$b
echo $c
c=$[$a+$b]
echo $c

[root@localhost sbin]# ./first.sh 
1+2
3
```

数学计算要用[]括起来，并且前面要加符号$。

## 和用户交互 

\


```
#!/bin/bash
#the name of this file is "first.sh"
## Using 'read' in shell script.
read -p "Please input a number : " x
read -p "Please input another number: " y
sum=$[$x+$y]
echo "The sum of the two numbers is: $sum"

[root@localhost sbin]# ./first.sh 
Please input a number : 2
Please input another number: 3
The sum of the two numbers is: 5
```

read命令用于和用户交互，它把用户输入的字符串作为变量值。

加上-x选项后可以查看执行的语句

```
[root@localhost sbin]# sh -x first.sh 
+ read -p 'Please input a number : ' x
Please input a number : 1
+ read -p 'Please input another number: ' y
Please input another number: 2
+ sum=3
+ echo 'The sum of the two numbers is: 3'
The sum of the two numbers is: 3
```

## shell 脚本预设变量 

```
#!/bin/bash
#the name of this file is "first.sh"
sum=$[$1+$2]
echo "sum=$sum"
echo "$0"


[root@localhost sbin]# sh -x first.sh 1 2
+ sum=3
+ echo sum=3
sum=3
+ echo first.sh
first.sh
```

本例中，$1和$2的值就是在执行时分别输入的1和2，$1就是脚本的第一个参数，$2是脚本的第二个参数，以此类推。一个shell脚本的预设变量是没有限制的。

另外还有一个$0，它代表脚本本身的名字。

# shell脚本里的逻辑判断

## if

格式：

```
if 判断语句1;then
commands
elif 判断语句2; then
commands
//多个elif
else
commands
fi


#!/bin/bash
#the name of this file is "first.sh"

read -p "Please input your score: " a
if (($a<60));then
echo "You didn't pass the exam."
elif (($a>=60));then
echo "you passed the exam!"
echo "congrats motherfucker"
else
echo "input a valid number your badass"
fi



[root@localhost sbin]# ./first.sh 
Please input your score: 59
You didn't pass the exam.
[root@localhost sbin]# ./first.sh 
Please input your score: 60
you passed the exam!
congrats motherfucker
[root@localhost sbin]# ./first.sh a
Please input your score: a
./first.sh:行5: ((: a: 表达式递归层次越界 （错误符号是 "a"）
./first.sh:行7: ((: a: 表达式递归层次越界 （错误符号是 "a"）
input a valid number your badass
```

上例中出现了(($a<60))这样的形式，这是shell脚本中特有的格式，只用一个小括号或者不用都会报错，请记住这个格式。这里a前面的$符号可以去掉

判断数值大小除了可以用(())的形式外，还可以使用[]。但是不能使用>、<、=这样的符号了，要使用-lt（小于)、-gt(大于)、-le(小于或等于)、-ge(大于或等于)、-eq(等于)、-ne(不等于)。下面就以命令行的形式简单比较一下，不再写shelI脚本。示例代码如下:

```
[root@localhost sbin]#  if [ 10 -gt 5 ];then echo ok;fi
ok
[root@localhost sbin]#  if [ 2 -gt 5 ] || [ 2 -lt 5 ];then echo ok;fi
ok
```

**注意：[]里面要有空格，将变量或常量与[]隔开**

### if和文档相关的判断

shell脚本中if还经常用于判断文档的属性，比如判断是普通文件还是目录，判断文件是否有读、写、执行权限等。if常用的选项有以下几个。

```
-e:判断文件或目录是否存在。
-d:判断是不是目录以及是否存在。
-f:判断是不是普通文件以及是否存在。
-r:判断是否有读权限。
-w:判断是否有写权限。
-x:判断是否可执行。
```

使用if判断时的具体格式如下:

```
if [ -e filename ] ; then
commands
fi


[root@localhost sbin]# if [ -d /home ]; then echo ok; fi
ok
[root@localhost sbin]# if [ -f /home ]; then echo ok; fi
[root@localhost sbin]# if [ -x first.sh ]; then echo ok; fi
ok
```

## case

```
case 变量 in 
value1)
	commands
  ;;
value2)
	commands
  ;;
value3)
	commands
  ;;
*)
	commands
  ;;
esac
```

＊代表其他值 

# shell 脚本中的循环 

## for

```
for 变量名 in循环的条件; do
		commands
done


[root@localhost sbin]# for i in 1 2 3 4 5; do echo $i;done
1
2
3
4
5
```

循环的条件还可以是一个命令的结果：

```
[root@localhost sbin]# for i in `seq 1 5`; do echo $i;done
1
2
3
4
5
[root@localhost sbin]# for i in `ls`; do echo $i;done
a1
a2
a3
a4
a5
first.sh
```

## while

while常用来编写死循环脚本，监控某项服务；

```
while 条件;do
		command
done

# cat while.sh
#!/bin/bash
a=5
while [ $a -ge 1 ]; do
		echo $a
		a=$[$a-1]
done

# sh while.sh
5
4
3
2
1
```

# shell 脚本中的中断和继续

很简单

```
if [ 判断条件 ];then
		break||continue||exit
fi
```

注：break退出当前层循环，continue进入当前层循环下一个阶段，exit直接退出整个脚本（相当于return）

# shell 脚本中的函数 

```
#cat func.sh
#!/bin/bash
function sum()
{
sum=$[$1+$2]
echo $sum
}

sum $1 $2
#sh func.sh 1 2
3
```

值得注意的是，在shell脚本中，函数一定要写在最前面，不能出现在中间或者最后。因为函数是要被调用的，如果还没有出现就被调用，肯定会出错。

#

大家帮点个赞，谢谢，创作不易！！！

