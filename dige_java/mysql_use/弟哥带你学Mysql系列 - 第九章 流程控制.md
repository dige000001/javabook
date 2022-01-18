## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

### 一. 分支函数

#### #1.if函数：if else 的效果

SELECT IF(10<5,'大','小');

SELECT 

last_name,commission_pct,IF(commission_pct IS NULL,'没奖金，呵呵','有奖金，嘻嘻') 备注

FROM 

employees;

#### #2.case函数

case 要判断的字段或表达式

when 常量1 then 要显示的值1或语句1

when 常量2 then 要显示的值2或语句2

...

else 要显示的值n或语句n

end

案例：查询员工的工资，要求

部门号=30，显示的工资为1.1倍

部门号=40，显示的工资为1.2倍

部门号=50，显示的工资为1.3倍

其他部门，显示的工资为原工资



SELECT salary 原始工资,department_id,

CASE department_id

WHEN 30 THEN salary*1.1

WHEN 40 THEN salary*1.2

WHEN 50 THEN salary*1.3

ELSE salary

END AS 新工资

FROM employees;

#案例：查询员工的工资的情况

如果工资>15000,显示B级别

如果工资>10000，显示C级别

否则，显示D级别



SELECT salary,

CASE 

WHEN salary>20000 THEN 'A'

WHEN salary>15000 THEN 'B'

WHEN salary>10000 THEN 'C'

ELSE 'D'

END AS 工资级别

FROM employees;

#案例1：

创建函数，实现传入成绩，如果成绩>90,返回A，如果成绩>80,返回B，如果成绩>60,返回C，否则返回D

CREATE FUNCTION test_case(score FLOAT) RETURNS CHAR

BEGIN 

DECLARE ch CHAR DEFAULT 'A';

CASE 

WHEN score>90 THEN SET ch='A';

WHEN score>80 THEN SET ch='B';

WHEN score>60 THEN SET ch='C';

ELSE SET ch='D';

END CASE;

RETURN ch;

END $




SELECT test_case(56)$

#### #3.if结构




语法：

if 条件1 then 语句1;

elseif 条件2 then 语句2;

....

else 语句n;

end if;

功能：类似于多重if

**只能应用在begin end 中**



#案例1：

创建函数，实现传入成绩，如果成绩>90,返回A，如果成绩>80,返回B，如果成绩>60,返回C，否则返回D

CREATE FUNCTION test_if(score FLOAT) RETURNS CHAR

BEGIN

DECLARE ch CHAR DEFAULT 'A';

IF score>90 THEN SET ch='A';

ELSEIF score>80 THEN SET ch='B';

ELSEIF score>60 THEN SET ch='C';

ELSE SET ch='D';

END IF;

RETURN ch;

END $



SELECT test_if(87)$

### #二、循环结构

分类：while、loop、repeat

#### 1.while

语法：

【标签:】while 循环条件 do

循环体;

end while【 标签】;


#案例：批量插入，根据次数插入到admin表中多条记录，如果次数>20则停止TRUNCATE TABLE admin$

DROP PROCEDURE test_while1$

CREATE PROCEDURE test_while1(IN insertCount INT)

BEGIN

DECLARE i INT DEFAULT 1;

a:WHILE i<=insertCount DO

INSERT INTO admin(username,`password`) VALUES(CONCAT('xiaohua',i),'0000');

IF i>=20 THEN LEAVE a;

END IF;

SET i=i+1;

END WHILE a;

END $

#### #2.loop/*


语法：

【标签:】loop

循环体;

end loop 【标签】;

可以用来模拟简单的死循环

#### #3.repeat



语法：

【标签：】repeat

循环体;

until 结束循环的条件

end repeat 【标签】;

