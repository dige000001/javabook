## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 基础查询 语法：

SELECT 要查询的东西 【FROM 表名】;



类似于Java中 :System.out.println(要打印的东西);

特点：

①通过select查询完的结果 ，是一个虚拟的表格，不是真实存在

② 要查询的东西 可以是常量值、可以是表达式、可以是字段、可以是函数

USE myemployees;

### #1.查询表中的单个字段



SELECT last_name FROM employees;



### #2.查询表中的多个字段

SELECT last_name,salary,email FROM employees;



### #3.查询表中的所有字段

#方式一：

SELECT 

`employee_id`,

`first_name`,

`last_name`,

`phone_number`,

`last_name`,

`job_id`,

`phone_number`,

`job_id`,

`salary`,

`commission_pct`,

`manager_id`,

`department_id`,

`hiredate` 

FROM

employees ;

#方式二： 

SELECT * FROM employees;

### #4.查询常量值

SELECT 100;

SELECT 'john';

### #5.查询表达式

SELECT 100%98;

### #6.查询函数

SELECT VERSION();

### #7.起别名

#方式一：使用as

SELECT 100%98 AS 结果;

SELECT last_name AS 姓,first_name AS 名 FROM employees;



#方式二：使用空格

SELECT last_name 姓,first_name 名 FROM employees;



对于内含关键字的别名，用双引号

SELECT salary AS "out put" FROM employees;

### #8.去重

#案例：查询员工表中涉及到的所有的部门编号

SELECT DISTINCT department_id FROM employees;

### #9.+号的作用

mysql中的+号：仅仅只有一个功能：运算符

select 100+90; 两个操作数都为数值型，则做加法运算

select '123'+90;只要其中一方为字符型，试图将字符型数值转换成数值型

如果转换成功，则继续做加法运算，该例结果为213

select 'john'+90; 如果转换失败，则将字符型数值转换成0

select null+10; 只要其中一方为null，则结果肯定为null

### #10.拼接

#案例：查询员工名和姓连接成一个字段，并显示为 姓名

SELECT CONCAT(字段1, 字段2, ……, 字段n) AS 结果;

SELECT 

CONCAT(last_name,first_name) AS 姓名

FROM

employees;

### #11.IFNULL

用法：

IFNULL(表达式1，表达式2)

若表达式1为NULL，则返回表达式2的值

SELECT 

IFNULL(commission_pct,0) AS 奖金率

FROM

employees;

# 条件查询

语法：

select 

查询列表

from

表名

where

筛选条件;



筛选条件分类：

一、按条件表达式筛选

简单条件运算符：> < = != <> >= <=

<> ：不等于

二、按逻辑表达式筛选

逻辑运算符：

&& || !

and or not




# 模糊查询

like

between and

in

is null
## 模糊查询

### like

特点：①一般和通配符搭配使用

**通配符：**

**%**  任意多个字符,包含0个字符

**_** 任意单个字符

#案例1：查询员工名中包含字符a的员工信息

select *

from

employees

where

last_name like '%a%';

#案例2：查询员工名中第三个字符为n，第五个字符为l的员工名和工资

select

last_name,

salary

FROM

employees

WHERE

last_name LIKE '__n_l%';

**#案例3：查询员工名中第二个字符为_的员工名**

SELECT

last_name

FROM

employees

WHERE

last_name LIKE '_$_%' ESCAPE '$';

ESCAPE表示该字符为转义字符

### BETWEEN AND

①使用between and 可以提高语句的简洁度

②包含临界值

③两个临界值不要调换顺序

#案例1：查询员工编号在100到120之间的员工信息

SELECT *

FROM

employees

WHERE

employee_id BETWEEN 120 AND 100

### IN

含义：判断某字段的值是否属于in列表中的某一项

特点：

①使用in提高语句简洁度

②in列表的值类型必须一致或兼容

③in列表中不支持通配符

#案例：查询员工的工种编号是 IT_PROG、AD_VP、AD_PRES中的一个员工名和工种编号

SELECT last_name, job_id

FROM

employees

WHERE

job_id **IN**( 'IT_PROT' ,'AD_VP','AD_PRES');

### IS NULL

=或<>不能用于判断null值

is null或is not null 可以判断null值

**<=>既可以判断数值又可以判断null值**

#案例1：查询有奖金的员工名和奖金率

SELECT

last_name,

commission_pct

FROM

employees

WHERE

commission_pct IS NOT NULL;

# 排序查询

语法：

select 查询列表

from 表名

【where 筛选条件】

order by 排序的字段或表达式;




特点：

1、asc代表的是升序，可以省略

desc代表的是降序

2、order by子句可以支持 单个字段、别名、表达式、函数、多个字段

3、order by子句在查询语句的最后面，除了limit子句

### 按别名排序

#案例：查询员工信息 按年薪升序

SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪

FROM employees

ORDER BY 年薪 ASC;




### 按函数排序

#案例：查询员工名，并且按名字的长度降序

SELECT LENGTH(last_name),last_name 

FROM employees

ORDER BY LENGTH(last_name) DESC;



### 按多个字段排序

#案例：查询员工信息，要求先按工资降序，再按employee_id升序

SELECT *

FROM employees

ORDER BY salary DESC,employee_id ASC;


# 常见函数

select 函数名(实参列表) 【from 表】;

分类：

**1、单行函数**

如 concat、length、ifnull等

**2、分组函数**

功能：做统计使用，又称为统计函数、聚合函数、组函数

## 单行函数

### 字符函数

#### #1.length 获取参数值的字节个数

SELECT LENGTH('john');

SELECT LENGTH('张三丰hahaha');




#### #2.concat 拼接字符串

SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;



#### #3.upper、lower

SELECT UPPER('john');

SELECT LOWER('joHn');

#示例：将姓变大写，名变小写，然后拼接

SELECT CONCAT(UPPER(last_name),LOWER(first_name)) 姓名 FROM employees;



#### #4.substr、substring

注意：索引从1开始

#截取从指定索引处后面所有字符

SELECT SUBSTR('李莫愁爱上了陆展元',7) out_put;



#截取从指定索引处指定字符长度的字符

SELECT SUBSTR('李莫愁爱上了陆展元',1,3) out_put;



#案例：姓名中首字符大写，其他字符小写然后用_拼接，显示出来



SELECT CONCAT(UPPER(SUBSTR(last_name,1,1)),'_',LOWER(SUBSTR(last_name,2))) out_put

FROM employees;


#### #5.instr 返回子串第一次出现的索引，如果找不到返回0


SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷八侠') AS out_put;




SELECT INSTR('杨不殷六侠悔爱上了殷六侠','殷六侠') AS out_put;




#### #6.trim

默认去除空格

SELECT TRIM(' 张翠山 ') AS out_put;

去除指定字符

SELECT TRIM('aa' FROM 'aaaaaaaaa张aaaaaaaaaaaa翠山aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa') AS out_put;




#### #7.lpad 用指定的字符实现左填充指定长度

SELECT LPAD('殷素素',9,'*') AS out_put;

若超过了指定长度，截断右边

SELECT LPAD('殷素素',2,'*') AS out_put;



#### #8.rpad 用指定的字符实现右填充指定长度

SELECT RPAD('殷素素',12,'ab') AS out_put;




#### #9.replace 替换

SELECT REPLACE('周芷若周芷若周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;

### 数学函数

#### #round 四舍五入

SELECT ROUND(-1.55);

精确到小数点后两位

SELECT ROUND(1.567,2); 结果 1.57




#### #ceil 向上取整,返回>=该参数的最小整数

SELECT CEIL(-1.02);



#### #floor 向下取整，返回<=该参数的最大整数

SELECT FLOOR(-9.99);




#### #truncate 截断

从小数点后一位开始截断

SELECT TRUNCATE(1.69999,1); 结果：1.6



#### #mod取余

**mod(a,b) ： a-a/b*b**




mod(-10,-3):-10- (-10)/(-3)*（-3）=-1

SELECT MOD(10,-3);

### 日期函数

#### #now 返回当前系统日期+时间

SELECT NOW();


#### #curdate 返回当前系统日期，不包含时间

SELECT CURDATE();




#### #curtime 返回当前时间，不包含日期

SELECT CURTIME();




#### #可以获取指定的部分，年、月、日、小时、分钟、秒

SELECT YEAR(NOW()) 年;

SELECT YEAR('1998-1-1') 年;

SELECT YEAR(hiredate) 年 FROM employees;

SELECT MONTH(NOW()) 月;

SELECT MONTHNAME(NOW()) 月;

#### #str_to_date 将字符通过指定的格式转换成日期
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/530a254e33834c278f4e26760f3a849a~tplv-k3u1fbpfcp-zoom-1.image)

SELECT STR_TO_DATE('1998-3-2','%Y-%c-%d') AS out_put;

SELECT STR_TO_DATE('3-2008-2','%m-%Y-%d') AS out_put;



#查询入职日期为1992--4-3的员工信息

SELECT * FROM employees WHERE hiredate = '1992-4-3';

SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');




#### #date_format 将日期转换成字符

**注，MYSQL中默认日期格式为：xxxx年xx月xx日**

SELECT DATE_FORMAT(NOW(),'%y年%m月%d日') AS out_put;



#查询有奖金的员工名和入职日期(xx月/xx日 xx年)

SELECT

last_name,DATE_FORMAT(hiredate,'%m月/%d日 %y年') 入职日期

FROM

employees

WHERE 

commission_pct IS NOT NULL;

#### #DATEDIFF 可用于计算日期差(天数)

SELECT DATEDIFF(MAX(hiredate),MIN(hiredate))FROM employees

#### #TimeStampdiff 可用于计算多种时间差

SELECT timestampdiff(hour,MAX(hiredate),MIN(hiredate))FROM employees;

时间差参数：

SECOND 秒

MINUTE 分钟

HOUR 小时

DAY 天

WEEK 星期

MONTH 月

QUARTER 季度

YEAR 年

### 其他函数

SELECT VERSION();

SELECT DATABASE();

SELECT USER();

###

## 分组函数(聚合函数)

功能：用作统计使用，又称为聚合函数或统计函数或组函数

分类：

sum 求和、avg 平均值、max 最大值 、min 最小值 、count 计算个数

特点：

1、 sum、avg一般用于处理数值型

max、min、count可以处理任何类型

以上分组函数都忽略null值

2、 可以和distinct搭配实现去重的运算

3、 count函数的单独介绍

一般使用count(*)用作统计行数

4、 和分组函数一同查询的字段要求是group by后的字段

#### 例子：

SELECT SUM(salary) FROM employees;SELECT AVG(salary) FROM employees;

SELECT MIN(salary) FROM employees;

SELECT MAX(salary) FROM employees;

SELECT COUNT(salary) FROM employees;




SELECT SUM(salary) 和,AVG(salary) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数

FROM employees;




SELECT SUM(salary) 和,ROUND(AVG(salary),2) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数

FROM employees;

#### #1、参数支持哪些类型

SELECT SUM(last_name) ,AVG(last_name) FROM employees; #禁止这么做

SELECT SUM(hiredate) ,AVG(hiredate) FROM employees;

字符串和日期可以比较：

SELECT MAX(last_name),MIN(last_name) FROM employees;

SELECT MAX(hiredate),MIN(hiredate) FROM employees;

SELECT COUNT(commission_pct) FROM employees;

SELECT COUNT(last_name) FROM employees;

#### #2、是否忽略null

SELECT 

SUM(commission_pct) ,AVG(commission_pct),SUM(commission_pct)/35,SUM(commission_pct)/107 

FROM

employees;

从上可以得出结论：SUM和AVG忽略了NULL值，NULL加任何东西都等于NULL

SELECT MAX(commission_pct) ,MIN(commission_pct) FROM employees;

SELECT COUNT(commission_pct) FROM employees;

SELECT commission_pct FROM employees;

#### #3、count函数的详细介绍

SELECT COUNT(*) FROM employees; //总行数，只要有一行某个值不为NULL，该行就会被计数

SELECT COUNT(1) FROM employees;//相当于多加一个字段，所有行该字段的值都为1，然后计算总行数

//即就算有一行全是NULL，也会被计算总行数

效率：

MYISAM存储引擎下 ，COUNT(*)的效率高

INNODB存储引擎下，COUNT(*)和COUNT(1)的效率差不多，比COUNT(字段)要高一些

#### #4、和分组函数一同查询的字段有限制

SELECT AVG(salary),employee_id FROM employees;

# 分组查询Group by

### 基础用法

语法：

select 分组函数, 字段

from 表

【where 筛选条件】

group by 分组的字段

【order by 排序的字段】;




用法：将表中数据分成若干组

GROUP BY会自动去重

特点：

1.  和分组函数一同查询的字段必须是group by后出现的字段

#案例1：查询每个工种的员工平均工资S

SELECT AVG(salary),job_id

FROM employees

GROUP BY job_id;

2.  筛选分为两类：分组前筛选和分组后筛选

分组前筛选：

数据源：原始表

语法：group by前 where

分组后筛选：

group by后的结果集

语法 ：group by后 having

#案例2：查询哪个部门的员工个数>5

SELECT department_id

FROM employees

GROUP BY department_id

HAVING COUNT(*)>5;

注：WHERE后不能有聚合函数，必须用HAVING

一般来讲，能用分组前筛选的，尽量使用分组前筛选，提高效率




3.  分组可以按单个字段也可以按多个字段
4.  可以搭配着排序使用

#案例：查询每个工种每个部门的最低工资,并按最低工资降序

SELECT MIN(salary),job_id,department_id

FROM employees

GROUP BY department_id,job_id

ORDER BY MIN(salary) DESC;




# 连接查询

按功能分类：

内连接：

等值连接

非等值连接

自连接

外连接：

左外连接

右外连接

全外连接

交叉连接

### SQL92的内连接

#### #1、等值连接

① 多表等值连接的结果为多表的交集部分②n表连接，至少需要n-1个连接条件

③ 多表的顺序没有要求

④一般需要为表起别名

⑤可以搭配前面介绍的所有子句使用，比如排序、分组、筛选

#查询员工名、工种号、工种名

SELECT e.last_name,e.job_id,j.job_title

FROM employees e,jobs j

WHERE e.`job_id`=j.`job_id`;

注意：如果为表起了别名，则查询的字段就不能使用原来的表名去限定



#### #2、非等值连接




#案例1：查询员工的工资和工资级别

SELECT salary,grade_level

FROM employees e,job_grades g

WHERE salary BETWEEN g.`lowest_sal` AND g.`highest_sal`

#### #3、自连接



#案例：查询 员工名和上级的名称

SELECT e.employee_id,e.last_name,m.employee_id,m.last_name

FROM employees e,employees m

WHERE e.`manager_id`=m.`employee_id`;

### SQL99的内连接

语法：

select 查询列表

from 表1 别名 

【连接类型】 join 表2 别名 

on 连接条件

【where 筛选条件】

【group by 分组】

【having 筛选条件】

【order by 排序列表】

连接类型：

分类：

内连接（★）：inner

外连接

左外(★):left 【outer】

右外(★)：right 【outer】

全外：full【outer】

交叉连接：cross 

#### 等值连接

语法：

select 查询列表

from 表1 别名

inner join 表2 别名

on 连接条件;



特点：

①添加排序、分组、筛选

②inner可以省略

③ 筛选条件放在where后面，连接条件放在on后面，提高分离性，便于阅读

④inner join连接和sql92语法中的等值连接效果是一样的，都是查询多表的交集

案例：查询有奖金的员工个数>3的部门名和员工个数和部门id，并按个数降序

SELECT COUNT(*) 个数,department_name,d.`department_id`

FROM employees e

INNER JOIN departments d

ON e.`department_id`=d.`department_id`

WHERE commission_pct IS NOT NULL

GROUP BY department_name,d.`department_id`

HAVING COUNT(*)>3

ORDER BY COUNT(*) DESC;

#### 非等值连接



#查询工资级别的个数>20的个数，并且按工资级别降序 

SELECT COUNT(*),grade_level

FROM employees e

JOIN job_grades g

ON e.`salary` BETWEEN g.`lowest_sal` AND g.`highest_sal`

GROUP BY grade_level

HAVING COUNT(*)>20

ORDER BY grade_level DESC;

#### 自连接


#查询姓名中包含字符k的员工的名字、上级的名字 

SELECT e.last_name,m.last_name

FROM employees e

JOIN employees m

ON e.`manager_id`= m.`employee_id`

WHERE e.`last_name` LIKE '%k%';

### 外连接

应用场景：用于查询一个表中有，另一个表没有的记录 

特点：

1.  外连接的查询结果为主表中的所有记录

如果从表中有和它匹配的，则显示匹配的值

如果从表中没有和它匹配的，则显示null

外连接查询结果=内连接结果+主表中有而从表没有的记录

2.  左外连接，left join左边的是主表

右外连接，right join右边的是主表

3.  左外和右外交换两个表的顺序，可以实现同样的效果 
3.  全外连接=内连接的结果+表1中有但表2没有的+表2中有但表1没有的

很遗憾，mysql不支持全连接



#案例1：查询哪个部门没有员工 #左外

SELECT d.*,e.employee_id

FROM departments d

LEFT OUTER JOIN employees e

ON d.`department_id` = e.`department_id`

WHERE e.`employee_id` IS NULL;

#交叉连接 实际是笛卡尔积

SELECT b.*,bo.*

FROM beauty b

CROSS JOIN boys bo;

# 子查询

含义：

出现在其他语句中的select语句，称为子查询或内查询

外部的查询语句，称为主查询或外查询



分类：

**按子查询出现的位置：**

select后面：

仅仅支持标量子查询

from后面：

支持表子查询

where或having后面：★

标量子查询（单行）★

列子查询 （多行）★

行子查询

exists后面（相关子查询）

表子查询

**按结果集的行列数不同：**

标量子查询（结果集只有一行一列）

列子查询（结果集只有一列多行）

行子查询（结果集有一行多列）

表子查询（结果集一般为多行多列）

## 一. WHERE后面的查询

1、标量子查询（单行子查询）

2、列子查询（多行子查询）

3、行子查询（多列多行）

特点：\
①子查询放在小括号内\
②子查询一般放在条件的右侧\
③标量子查询，一般搭配着单行操作符（> < >= <= = <>）使用

列子查询，一般搭配着多行操作符（in、any/some、all）使用

④子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果

### #1.标量子查询★

#案例1：查询最低工资大于50号部门最低工资的部门id和其最低工资

SELECT MIN(salary),department_id

FROM employees

GROUP BY department_id

HAVING MIN(salary)>(

SELECT MIN(salary)

FROM employees

WHERE department_id = 50

);

### #2.列子查询（多行子查询）★

#案例3：返回其它部门中比job_id为‘IT_PROG’部门所有工资都低的员工 的员工号、姓名、job_id 以及salary

SELECT last_name,employee_id,job_id,salary

FROM employees

WHERE salary<ALL(

SELECT DISTINCT salary

FROM employees

WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

### #3、行子查询（结果集一行多列或多行多列）

#案例：查询员工编号最小并且工资最高的员工信息

SELECT * 

FROM employees

WHERE (employee_id,salary)=(

SELECT MIN(employee_id),MAX(salary)

FROM employees

);

## 二. select后面

仅仅支持标量子查询

#案例：查询每个部门的员工个数

SELECT d.*,(

SELECT COUNT(*)

FROM employees e

WHERE e.department_id = d.`department_id`

) 个数

FROM departments d;

## 三、from后面

将子查询结果充当一张表，要求必须起别名

#案例：查询每个部门的平均工资的工资等级

SELECT ag_dep.*,g.`grade_level`

FROM (

SELECT AVG(salary) ag,department_id

FROM employees

GROUP BY department_id

) ag_dep

INNER JOIN job_grades g

ON ag_dep.ag BETWEEN lowest_sal AND highest_sal;

## 四、exists后面（相关子查询）

**语法**：exists(完整的查询语句)

结果：1或0

#案例1：查询有员工的部门名

SELECT department_name

FROM departments d

WHERE EXISTS(

SELECT *

FROM employees e

WHERE d.`department_id`=e.`department_id`

);

# 分页查询

应用场景：当要显示的数据，一页显示不全，需要分页提交

**sql请求语法：**

select 查询列表

from 表

【join type】 join 表2

on 连接条件

where 筛选条件

group by 分组字段

having 分组后的筛选

order by 排序的字段】

limit 【offset,】size;

offset要显示条目的起始索引（起始索引从0开始）

size 要显示的条目个数

limit语句放在查询语句的最后

案例3：有奖金的员工信息，并且工资较高的前10名显示出来

SELECT * 

FROM employees 

WHERE commission_pct IS NOT NULL 

ORDER BY salary DESC 

LIMIT 10 ;


# 联合查询

union：将多条查询语句的结果合并成一个结果

**语法：**

查询语句1

union

查询语句2

union

**应用场景：** 要查询的结果来自于多个表，且多个表没有直接的连接关系，但**查询的信息一致**时

**特点：** ★

1、要求多条查询语句的查询列数是一致的！

2、要求多条查询语句的查询的每一列的类型和顺序最好一致

3、union关键字**默认去重**，如果使用**union all** 可以包含重复项

查询部门编号>90或邮箱包含a的员工信息

SELECT * FROM employees WHERE email LIKE '%a%'

UNION

SELECT * FROM employees WHERE department_id>90;

Select语句的字段名可以不同，以第一条语句为准显示信息

