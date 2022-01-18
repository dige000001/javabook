## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

**#常见的数据类型/***

数值型：

整型

小数：

定点数

浮点数

字符型：

较短的文本：char、varchar

较长的文本：text、blob（较长的二进制数据）



日期型：

datetime保存日期+时间

timestamp保存日期+时间


# #一、整型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c540687e13cb48648a9543a9f16978bd~tplv-k3u1fbpfcp-zoom-1.image)




特点：

① 如果不设置无符号还是有符号，默认是有符号，如果想设置无符号，需要添加unsigned关键字

② 如果插入的数值超出了整型的范围,会报out of range异常，并且插入临界值

③ 如果不设置长度，会有默认的长度

**长度代表了显示的最大宽度，如果不够会用0在左边填充，但必须搭配zerofill使用！**



# 1、小数




**分类：**

1.浮点型

float(M,D)

double(M,D)

2.定点型

dec(M，D)

decimal(M,D)




**特点：**

①

M：整数部位+小数部位

D：小数部位

如果超过范围，则插入临界值



②

M和D都可以省略

如果是decimal，则M默认为10，D默认为0

如果是float和double，则会根据插入的数值的精度来决定精度




③定点型的精确度较高，如果要求插入数值的精度较高如货币运算等则考虑使用



#测试M和D

DROP TABLE tab_float;

CREATE TABLE tab_float(

f1 FLOAT,

f2 DOUBLE,

f3 DECIMAL

);

SELECT * FROM tab_float;

DESC tab_float;



INSERT INTO tab_float VALUES(123.4523,123.4523,123.4523);

INSERT INTO tab_float VALUES(123.456,123.456,123.456);

INSERT INTO tab_float VALUES(123.4,123.4,123.4);

INSERT INTO tab_float VALUES(1523.4,1523.4,1523.4);




# 2、字符型

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fdb3db28357b43348194a3d9da4972ad~tplv-k3u1fbpfcp-zoom-1.image)

其他：

binary和varbinary用于保存较短的二进制

enum用于保存枚举

set用于保存集合



CREATE TABLE tab_char(

c1 ENUM('a','b','c')

);



INSERT INTO tab_char VALUES('a');

INSERT INTO tab_char VALUES('b');

INSERT INTO tab_char VALUES('c');

INSERT INTO tab_char VALUES('m');

INSERT INTO tab_char VALUES('A');




SELECT * FROM tab_set;



CREATE TABLE tab_set(

s1 SET('a','b','c','d')

);

INSERT INTO tab_set VALUES('a');

INSERT INTO tab_set VALUES('A,B');

INSERT INTO tab_set VALUES('a,c,d');



# 3、日期型


分类：

date只保存日期

time 只保存时间

year只保存年


datetime保存日期+时间

timestamp保存日期+时间



![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ba5113e5f2b47a68cd007fde8efe295~tplv-k3u1fbpfcp-zoom-1.image)



CREATE TABLE tab_date(

t1 DATETIME,

t2 TIMESTAMP

);




INSERT INTO tab_date VALUES(NOW(),NOW());

SELECT * FROM tab_date;


