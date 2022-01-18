---
## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

存储过程和函数：类似于java中的方法

好处：

1、提高代码的重用性

2、简化操作



# #存储过程

含义：一组预先编译好的SQL语句的集合，理解成批处理语句

1、提高代码的重用性

2、简化操作

**3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率**

## #一、创建语法

CREATE PROCEDURE 存储过程名(参数列表)

BEGIN

存储过程体（一组合法的SQL语句）

END

#注意：

**1、参数列表包含三部分**

参数模式 参数名 参数类型

举例：

in stuname varchar(20)



**参数模式：**

in：该参数可以作为输入，也就是该参数需要调用方传入值

out：该参数可以作为输出，也就是该参数可以作为返回值

inout：该参数既可以作为输入又可以作为输出，也就是该参数既需要传入值，又可以返回值




**2、如果存储过程体仅仅只有一句话，begin end可以省略**

**3、存储过程体中的每条sql语句的结尾要求必须加分号。**

**4、存储过程的结尾可以使用 delimiter 重新设置**

语法：

delimiter 结束标记

案例：

delimiter $

设置结束符的原因：存储过程是一个代码段，在mysql执行过程中，遇到分号就执行了。有些情况分号不代表语句结束，故需要重新设置结束符。

## #二、调用语法

CALL 存储过程名(实参列表);

**#--------------------------------案例演示-----------------------------------#**

### 1.空参列表

#案例：插入到beauty表中五条记录


DELIMITER $

CREATE PROCEDURE myp1()

BEGIN

SELECT bo.*

FROM boys bo

RIGHT JOIN beauty b ON bo.id = b.boyfriend_id

WHERE b.name='beautyName';

END $

### 2.创建带in模式参数的存储过程

#案例1：创建存储过程实现 根据女神名，查询对应的男神信息


DELIMITER $

CREATE PROCEDURE myp2(IN beautyName VARCHAR(20))

BEGIN

SELECT bo.*

FROM boys bo

RIGHT JOIN beauty b ON bo.id = b.boyfriend_id

WHERE b.name=beautyName;

END $



#调用

CALL myp2('柳岩')$

### #3.创建out 模式参数的存储过程



#案例2：根据输入的女神名，返回对应的男神名和魅力值

CREATE PROCEDURE myp7(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20),OUT usercp INT) 

BEGIN

SELECT boys.boyname ,boys.usercp INTO boyname,usercp

FROM boys 

RIGHT JOIN

beauty b ON b.boyfriend_id = boys.id

WHERE b.name=beautyName ;

END $

CALL myp7('小昭',@name,@cp)$

SELECT @name,@cp$

### #4.创建带inout模式参数的存储过程

#案例1：传入a和b两个值，最终a和b都翻倍并返回

CREATE PROCEDURE myp8(INOUT a INT ,INOUT b INT)

BEGIN

SET a=a*2;

SET b=b*2;

END $

#调用

SET @m=10$

SET @n=20$

CALL myp8(@m,@n)$

SELECT @m,@n$

# #三、删除存储过程

#语法：drop procedure 存储过程名

DROP PROCEDURE p1;

DROP PROCEDURE p2,p3;#×


# #四、查看存储过程的信息

DESC myp2;×

SHOW CREATE PROCEDURE myp2;

# 函数

含义：一组预先编译好的SQL语句的集合，理解成批处理语句

1、提高代码的重用性

2、简化操作

3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率



**区别：**

**存储过程：可以有0个返回，也可以有多个返回，适合做批量插入、批量更新**

**函数：有且仅有1 个返回，适合做处理数据后返回一个结果**



## #一、创建语法

CREATE FUNCTION 函数名(参数列表) RETURNS 返回类型

BEGIN

函数体

END




注意：

1.参数列表 包含两部分：

参数名 参数类型




2.函数体：肯定会有return语句，如果没有会报错

如果return语句没有放在函数体的最后也不报错，但不建议




return 值;

3.函数体中仅有一句话，则可以省略begin end

4.使用 delimiter语句设置结束标记






#案例2：根据部门名，返回该部门的平均工资



CREATE FUNCTION myf3(deptName VARCHAR(20)) RETURNS DOUBLE

BEGIN

DECLARE sal DOUBLE ;

SELECT AVG(salary) INTO sal

FROM employees e

JOIN departments d ON e.department_id = d.department_id

WHERE d.department_name=deptName;

RETURN sal;

END $



SELECT myf3('IT')$

#案例3、创建函数，实现传入两个float，返回二者之和




CREATE FUNCTION test_fun1(num1 FLOAT,num2 FLOAT) RETURNS FLOAT

BEGIN

DECLARE SUM FLOAT DEFAULT 0;

SET SUM=num1+num2;

RETURN SUM;

END $



SELECT test_fun1(1,2)$


#三、查看函数




SHOW CREATE FUNCTION myf3;




#四、删除函数

DROP FUNCTION myf3;


