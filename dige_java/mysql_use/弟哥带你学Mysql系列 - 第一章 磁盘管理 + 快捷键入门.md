## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 基础命令

在环境变量中添加mysql文件夹的bin目录后：

1.  登入mysql

mysql -u 用户名 -p

密码

2.  展示所有数据库

show databases;

3.  打开某个数据库

use 数据库名

4.  展示库里的表

show tables;

或者

show tables from 数据库名;

5.  查看当前使用的数据库

select database();

6.  建表

creat table 表名 (

属性1 类型,

属性2 类型,

……

);

7.  查看某张表的结构

desc 表名;

8.  查看mysql版本

在cmd 里 

mysql --version

或在mysql里

select version()；

# Mysql语法规范

1.  不区分大小写，但建议关键字大写，表名，列名小写
1.  每条命令建议分号结尾

<!---->

3.  注释

单行注释：#

多行：/* */

