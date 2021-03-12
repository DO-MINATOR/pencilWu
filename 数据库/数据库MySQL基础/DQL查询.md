### 概览

![在这里插入图片描述](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/20200703100542907.png)

### 基础查询

`select 字段 from 表名；`

```mysql
USE myemployees;

#1.查询表中的单个字段
SELECT last_name FROM employees;

#2.查询表中多个字段
SELECT last_name,salary,email FROM employees;

#3.查询表中的所有字段
SELECT * FROM employees;

#4.起别名
#方式一:使用AS
SELECT last_name AS 姓,first_name AS 名 FROM employees;

#方式二:使用空（推荐）
SELECT last_name 姓,first_name 名 FROM employees;

#案例:查询salary,结果显示 out put
SELECT salary AS "out put" FROM employees;

#5.去重
#案例:查询员工表中涉及的所有部门编号
SELECT DISTINCT department_id FROM employees;

#6.+号的作用
#【错误】查询员工的名和姓,并显示为姓名
/*
select 100+90; 两个操作数都为数值型,做加法运算
select '123'+90;其中一方为字符型,试图将字符型数值转换为数值型
select 'john'+90; 如果转换失败,则将字符型数值转换成0
select null+0; 只要其中一方为null,则结果肯定为null.
*/
SELECT last_name+first_name AS 姓名 FROM employees; ×

#7.【正确】concat函数 
/*
功能：拼接字符
select concat(字符1，字符2，字符3,...);
*/
SELECT CONCAT('a','b','c') 结果 FROM employees;
SELECT CONCAT(last_name,first_name) 姓名 FROM employees;

#8.【补充】ifnull函数
#功能：判断某字段或表达式是否为null，如果为null 返回指定的值，否则返回原本的值

SELECT IFNULL(commission_pct,0) FROM employees;

#9.【补充】isnull函数
#功能：判断某字段或表达式是否为null，如果是，则返回1，否则返回0
```

### 条件查询

`select 字段 from 表名 where 条件；`

1. 按条件表达式筛选

   条件运算符:> < = != <> >= <= <=>安全等于。

2. 按逻辑表达式筛选

   逻辑运算符:&& || !	and or not

3. 模糊查询

   like:一般搭配通配符使用，%任意多个字符，_任意单个字符

   between and

   in

   is null

```mysql
#一.按条件表达式筛选
#案例1:查询工资>12000的员工信息
SELECT * FROM employees WHERE salary>12000;

#案例2:查询部门编号不等于90号的员工名和部门编号
SELECT last_name,department_id FROM employees WHERE department_id <> 90;

#二、按逻辑表达式筛选
#案例1:查询工资z在10000到20000之间的员工名、工资及奖金
SELECT last_name,salary,commission_pct FROM employees WHERE salary>=10000 AND salary<=20000;

#案例2:查询部门编号不是在90-110之间,或者工资高于15000的员工信息
SELECT * FROM employees WHERE department_id <90 OR department_id>110 OR salary>15000;

#三、模糊查询
#1.like适用于字符型和数值型
#案例1:查询员工名中包含字符a的员工信息
SELECT * FROM employees WHERE last_name LIKE '%a%';

#案例2:查询员工名中第三个字符为b，第五个字符为a的员工名和工资
SELECT last_name,salary FROM employees WHERE last_name LIKE '__b_a%';

#案例3:查询员工名种第二个字符为_的员工名
SELECT last_name FROM employees WHERE last_name LIKE '_\_%';

#2.between and
#案例1:查询员工编号在100到120之间的员工信息
SELECT * FROM employees WHERE employee_id BETWEEN 100 AND 120;

#3.in
#案例1:查询员工的工种编号是IT_PROG、AD_VP、AD_PRES中的一个员工名和工种编号
SELECT last_name,job_id FROM employees WHERE job_id='IT_PROG' OR job_id='AD_PRES' OR job_id='AD_VP';
SELECT last_name,job_id FROM employees WHERE job_id IN('IT_PROG','AD_PRES','AD_VP');

#4.is null
/*
=或<>不能用于判断null值
is null 或 is not null 可以判断null值
*/
#案例1:查询没有奖金的员工名和奖金率
SELECT last_name,commission_pct FROM employees WHERE commission_pct IS NULL;
SELECT last_name,commission_pct FROM employees WHERE commission_pct IS NOT NULL;

#安全等于<=>,这个相比=或<>可以判断为null的情况
#案例1:查询没有奖金的员工名和奖金率
SELECT last_name,commission_pct FROM employees WHERE commission_pct <=> NULL;
#案例2:查询工资为12000的员工信息
SELECT last_name,commission_pct FROM employees WHERE salary <=> 12000;
```

### 排序查询

`select 字段 from 表名 where 条件 oreder by 字段；`

- asc（默认）升序，desc降序
- order by支持多个字段组合操作
- 一般作为语句的最后部分，除了limit

```mysql
#案例1:查询员工信息,要求工资排序
SELECT * FROM employees ORDER BY salary DESC;#降序
SELECT * FROM employees ORDER BY salary;#升序

#案例2:查询部门编号是>=90，按入职时间的先后进行排序
SELECT * FROM employees WHERE department_id>=90 ORDER BY hiredate ASC;

#案例3:按年薪的高低显示员工的信息和年薪【按表达式排序】
SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪 FROM employees 
ORDER BY salary*12*(1+IFNULL(commission_pct,0)) DESC; 

#案例4:按年薪的高低显示员工的信息和年薪【按别名排序】
SELECT *,salary*12*(1+IFNULL(commission_pct,0)) 年薪 FROM employees 
ORDER BY 年薪 DESC; 

#案例5:按姓名的长度显示员工的姓名和工资【按函数排序】
SELECT LENGTH(last_name) 字节长度,last_name,salary FROM employees
ORDER BY LENGTH(last_name) DESC;

#案例6:查询员工共信息,要求按工资排序，再按员工编号排序【按多个字段排序】
SELECT * FROM employees
ORDER BY salary ASC,employee_id DESC;
```

### 函数

`select 函数名(字段) from 表名；`

对选取的字段做一定处理，并显示该结果

- 单行函数：CONCAT、LENGTH、IFNULL，传入单个参数，并返回一个结果
- 分组函数：做统计使用

#### 单行函数

**字符函数：**对查询结果的字符做处理

```mysql
#一.字符函数
#1.length 获取参数值的字节值
SELECT LENGTH('subei');
SELECT LENGTH('鬼谷子qwe');

#2.concat 拼接字符串
SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;

#3.upper:变大写、lower：变小写

SELECT UPPER('ton');
SELECT LOWER('ton');

#示例：将姓变大写，名变小写，然后拼接
SELECT CONCAT(UPPER(last_name),LOWER(first_name)) 姓名 FROM employees;

#4.substr
#注意:索引从1开始
#截取从指定所有处后面的所以字符
SELECT SUBSTR('吴刚伐桂在天上',4) out_put;

#截取从指定索引处指定字符长度的字符
SELECT SUBSTR('吴刚伐桂在天上',1,2) out_put;

#案例:姓名中首字符大写,其他字符小写，然后用_拼接,显示出来
SELECT CONCAT(UPPER(SUBSTR(last_name,1,1)),'_',LOWER(SUBSTR(last_name,2))) out_put FROM employees;

#5.instr:获取子串第一次出现的索引,找不到返回0
SELECT INSTR('MySQL技术进阶','技术') AS out_put;

#6.trim:去前后空格
SELECT LENGTH(TRIM('	霍山	')) AS out_put;
#去掉指定字符
SELECT TRIM('+' FROM '++++李刚+++刘邦+++') AS out_put;

#7.lpad:用指定的字符实现左填充指定长度
SELECT LPAD('梅林',8,'+') AS out_put;
#8.rpad:用指定的字符实现右填充指定长度
SELECT RPAD('梅林',5,'&') AS out_put;

#9.replace:替换
SELECT REPLACE('莉莉伊万斯的青梅竹马是詹姆','詹姆','斯内普') AS out_put;
```

**数学函数：**对查询结果的数值做处理

```mysql
#1.round:四舍五入
SELECT ROUND(1.45);
SELECT ROUND(1.567,2);

#2.ceil:向上取整,返回>=该参数的最小整数
SELECT CEIL(1.005);
SELECT CEIL(-1.002);

#3.floor:向下取整,返回<=该参数的最大整数
SELECT FLOOR(-9.99);

#4.truncate:小数点后数位截断
SELECT TRUNCATE(1.65,1);

#5.mod:取余，符号与被除数相同
SELECT MOD(10,3);

#6.rand:获取随机数，返回0-1之间的小数
SELECT RAND();
```

**日期函数：**

```mysql
#1.now:返回当前日期+时间
SELECT NOW();

#2.year:返回年
SELECT YEAR(NOW());
SELECT YEAR(hiredate) 年 FROM employees;

#3.month:返回月
#MONTHNAME:以英文形式返回月
SELECT MONTH(NOW());
SELECT MONTHNAME(NOW());

#4.day:返回日
#DATEDIFF:返回两个日期相差的天数
SELECT DAY(NOW());
SELECT DATEDIFF('2020/06/30','2020/06/21');

#5.str_to_date:将字符通过指定格式转换成日期
SELECT STR_TO_DATE('2020-5-13','%Y-%c-%d') AS out_put;

#6.date_format:将日期转换成字符
SELECT DATE_FORMAT('2020/6/6','%Y年%m月%d日') AS out_put;
SELECT DATE_FORMAT(NOW(),'%Y年%m月%d日') AS out_put;

#7.curdate:返回当前日期
SELECT CURDATE();

#8.curtime:返回当前时间
SELECT CURTIME();
```

**其他：**

```mysql
#version 当前数据库服务器的版本
SELECT VERSION();

#database 当前打开的数据库
SELECT DATABASE();

#user当前用户
SELECT USER();

#md5('字符'):返回该字符的md5加密形式
SELECT MD5('a');
```

**流程控制函数:**

```mysql
#1.if函数: if else效果
SELECT IF(10<5,'大','小');
SELECT last_name,commission_pct,IF(commission_pct IS NULL,'没奖金！！！','有奖金!!!') 备注 FROM employees;

#2.case函数，两种写法
SELECT salary 原始工资,department_id,
CASE department_id
WHEN 30 THEN salary*1.1
WHEN 40 THEN salary*1.2
WHEN 50 THEN salary*1.3
ELSE salary
END AS 新工资
FROM employees;

SELECT salary,
CASE
WHEN salary>20000 THEN 'A'
WHEN salary>15000 THEN 'B'
WHEN salary>10000 THEN 'C'
ELSE 'D'
END AS 工资等级
FROM employees;
```

#### 分组函数

又称作统计函数、组函数

sum 求和、avg 平均值、max 最大值、min最小值count 计算个数，且可以和distinct函数搭配实现去重操作，count(字段)统计该字段非空的个数，count(*)统计所有存在的行数

```mysql
#1.简单使用
SELECT SUM(salary) 和,ROUND(AVG(salary),2) 平均,MAX(salary) 最高,MIN(salary) 最低,COUNT(salary) 个数
FROM employees;

#2.参数支持哪些数据类型
SELECT SUM(last_name),AVG(last_name) FROM employees;
SELECT SUM(hiredate),AVG(hiredate) FROM employees;

SELECT MAX(last_name),MIN(last_name) FROM employees;
SELECT MAX(hiredate),MIN(hiredate) FROM employees;

SELECT COUNT(commission_pct) FROM employees;
SELECT COUNT(last_name) FROM employees;

#3.是否忽略null（是）
SELECT 
    SUM(commission_pct),
    AVG(commission_pct),
    SUM(commission_pct) / 35,
    SUM(commission_pct) / 107
FROM
    employees;

#4.和distinct搭配
SELECT SUM(DISTINCT salary),SUM(salary) FROM employees;
SELECT COUNT(DISTINCT salary),COUNT(salary) FROM employees;

#5.count函数详解
SELECT COUNT(salary) FROM employees;
SELECT COUNT(*) FROM employees;
SELECT COUNT(1) FROM employees;#新增‘1’列
```

### 分组查询

```mysql
select 分组函数,字段a
from 表
【where 筛选条件】
group by 字段a
【having 分组后的筛选】
【order by 排序列表】
```

|                | 关键字 | 筛选表     | 位置       |
| -------------- | ------ | ---------- | ---------- |
| 分组查询前筛选 | WHERE  | 原始表     | GROUP BY前 |
| 分组查询后筛选 | HAVING | 分组后的表 | GROUP BY后 |

group by支持多字段分组

```mysql
#引入:查询每个部门的平均工资
SELECT AVG(salary) FROM employees;

#案例1:查询每个工种的最高工资
SELECT MAX(salary),job_id FROM employees 
GROUP BY job_id;

#案例2:查询每个位置上的部门个数
SELECT COUNT(*),location_id
FROM departments
GROUP BY location_id;

#添加筛选条件
#案例1:查询邮箱中包含a字符的，每个部门的平均工资
SELECT AVG(salary),department_id FROM employees
WHERE email LIKE '%a%' GROUP BY department_id;

#案例2:查询有奖金的每个领导手下员工的最高工资
SELECT MAX(salary),manager_id FROM employees
WHERE commission_pct IS NOT NULL
GROUP BY manager_id;

#添加复杂的筛选条件
#案例1:查询哪个部门的员工个数>2
SELECT COUNT(*) sum,department_id FROM employees
GROUP BY department_id HAVING sum >2;

#案例2:查询每个工种有奖金的员工的最高工资>12000的工种编号和最高工资 
SELECT MAX(salary), job_id FROM employees 
WHERE commission_pct IS NOT NULL GROUP BY job_id 
HAVING MAX(salary)>12000; 

#按表达式或函数分组
#案例:按员工姓名的长度分组,查询每一组的员工个数,筛选员工个数>5
SELECT COUNT(*) c,LENGTH(last_name) len_name 
FROM employees GROUP BY len_name HAVING c>5;

#按多个字段查询
#案例:查询每个部门每个工种的员工的平均工资
SELECT AVG(salary),department_id,job_id
FROM employees GROUP BY department_id,job_id;

#添加排序
#案例:查询每个部门每个工种的员工的平均工资,按平均工资的高低查询
SELECT AVG(salary),department_id,job_id
FROM employees GROUP BY department_id,job_id
ORDER BY AVG(salary) DESC;
```

### 连接查询

又称多表查询，当待查询字段来自多个表时，就用该种方式，笛卡尔乘积现象，交叉连接。

```
内连接(inner join)：
	等值连接
	非等值连接
	自连接
外连接：
	左外连接(left join)
	右外连接(right join)
	全外连接(full join)
交叉连接：(cross join)
```
#### sql99

```mysql
内连接
/*
语法：
select 查询列表
from 表1 别名
inner join 表2 别名
on 连接条件;

特点：
①添加排序、分组、筛选
②inner可以省略
③筛选条件放在where后面，连接条件放在on后面，提高分离性，便于阅读
*/
等值连接
#查询员工名、部门名
SELECT last_name,department_name FROM departments d
INNER JOIN  employees e
ON e.`department_id` = d.`department_id`;

#查询员工名、部门名、工种名，并按部门名降序（添加三表连接）
SELECT last_name,department_name,job_title
FROM employees e
INNER JOIN departments d ON e.`department_id`=d.`department_id`
INNER JOIN jobs j ON e.`job_id` = j.`job_id`
ORDER BY department_name DESC;

非等值连接
#查询员工的工资级别
SELECT salary,grade_level
FROM employees e
INNER JOIN job_grades g
ON e.`salary` BETWEEN g.`lowest_sal` AND g.`highest_sal`;

自连接 
#查询员工的名字、上级的名字
SELECT e.last_name,m.last_name
FROM employees e
INNER JOIN employees m
ON e.`manager_id`= m.`employee_id`;
```

```mysql
外连接
/*
应用场景：用于查询一个表中有，另一个表没有的记录
特点：
1、外连接的查询结果为主表中的所有记录
	如果从表中有和它匹配的，则显示匹配的值
	如果从表中没有和它匹配的，则显示null
	外连接查询结果=内连接结果+主表中有而从表没有的记录
2、左外连接，left join左边的是主表
   右外连接，right join右边的是主表
3、左外和右外交换两个表的顺序，可以实现同样的效果 
4、全外连接=内连接的结果+表1中有但表2没有的+表2中有但表1没有的
*/
#查询哪个部门没有员工
#左外
SELECT 
    d.*, e.employee_id
FROM
    departments d
        LEFT JOIN
    employees e ON d.`department_id` = e.`department_id`
WHERE
    e.`employee_id` IS NULL;

#右外
SELECT 
    d.*, e.employee_id
FROM
    employees e
        RIGHT JOIN
    departments d ON d.`department_id` = e.`department_id`
WHERE
    e.`employee_id` IS NULL;

#全外(内连接+左连接+右连接)
SELECT b.*,bo.* FROM beauty b
FULL JOIN boys bo
ON b.`boyfriend_id` = bo.id;

#交叉连接（笛卡儿积）
SELECT b.*,bo.* FROM beauty b
CROSS JOIN boys bo;
```

### 子查询

出现在其他语句中的select语句,称为子查询或内查询外部的查询语句，称为主查询或外查询。

![在这里插入图片描述](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/20200703113430918.png)

```mysql
#一、where或having后面
/*
特点：
①子查询放在小括号内
②子查询一般放在条件的右侧
③标量子查询，一般搭配着单行操作符使用
    > < >= <= = <>
④列子查询，一般搭配着多行操作符使用
    in、any/some、all
⑤子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果
*/

标量子查询
#案例1：谁的工资比 Abel 高?
#①查询Abel的工资
SELECT salary
FROM employees
WHERE last_name = 'Abel';
#②查询员工的信息，满足 salary>①结果
SELECT *
FROM employees
WHERE salary>(
	SELECT salary
	FROM employees
	WHERE last_name = 'Abel'
);

#案例2：返回job_id与141号员工相同，salary比143号员工多的员工姓名，job_id和工资
SELECT last_name,job_id,salary
FROM employees
WHERE job_id = (
	SELECT job_id
	FROM employees
	WHERE employee_id = 141
) AND salary>(
	SELECT salary
	FROM employees
	WHERE employee_id = 143
);

#案例3：返回公司工资最少的员工的last_name,job_id和salary
SELECT last_name,job_id,salary
FROM employees
WHERE salary=(
	SELECT MIN(salary)
	FROM employees
);

#案例4：查询最低工资大于50号部门最低工资的部门id和其最低工资
#①查询50号部门的最低工资
SELECT  MIN(salary)
FROM employees
WHERE department_id = 50;

#②查询每个部门的最低工资
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id;

#③在②基础上筛选，满足min(salary)>①
SELECT MIN(salary),department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary)>(
	SELECT  MIN(salary)
	FROM employees
	WHERE department_id = 50
);

列子查询
#案例1：返回location_id是1400或1700的部门中的所有员工姓名
SELECT last_name
FROM employees
WHERE department_id in(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id IN(1400,1700)
);

#案例2：返回其它工种中比job_id为‘IT_PROG’工种任一工资低的员工的员工号、姓名、job_id 以及salary
#①查询job_id为‘IT_PROG’部门任一工资
SELECT DISTINCT salary FROM employees
WHERE job_id = 'IT_PROG';

#②查询员工号、姓名、job_id 以及salary，salary<(①)的任意一个
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<ANY(
	SELECT DISTINCT salary
	FROM employees
	WHERE job_id = 'IT_PROG'

) AND job_id<>'IT_PROG';

#或
SELECT last_name,employee_id,job_id,salary
FROM employees
WHERE salary<(
	SELECT MAX(salary)
	FROM employees
	WHERE job_id = 'IT_PROG'
) AND job_id<>'IT_PROG';

行子查询（结果集一行多列或多行多列）

#案例：查询员工编号最小并且工资最高的员工信息
SELECT * FROM employees
WHERE (employee_id,salary)=(
	SELECT MIN(employee_id),MAX(salary)
	FROM employees
);
#①查询最小的员工编号
SELECT MIN(employee_id) FROM employees;
#②查询最高工资
SELECT MAX(salary) FROM employees;
#③查询员工信息
SELECT * FROM employees
WHERE employee_id=(
	SELECT MIN(employee_id)
	FROM employees
)AND salary=(
	SELECT MAX(salary)
	FROM employees
);
```

```mysql
#二、select后面
/*
仅仅支持标量子查询
*/
#案例：查询每个部门的员工个数
SELECT d.*,(
	SELECT COUNT(*)
	FROM employees e
	WHERE e.department_id = d.`department_id`
 ) 个数
 FROM departments d;
 
#案例2：查询员工号=102的部门名
SELECT (
	SELECT department_name,e.department_id
	FROM departments d
	INNER JOIN employees e
	ON d.department_id=e.department_id
	WHERE e.employee_id=102
) 部门名;
```

```mysql
#三、from后面
/*
将子查询结果充当一张表，要求必须起别名
*/
#案例：查询每个部门的平均工资的工资等级
#①查询每个部门的平均工资
SELECT AVG(salary),department_id
FROM employees GROUP BY department_id;

SELECT * FROM job_grades;

#②连接①的结果集和job_grades表，筛选条件平均工资 between lowest_sal and highest_sal
SELECT  ag_dep.*,g.`grade_level`
FROM (
	SELECT AVG(salary) ag,department_id
	FROM employees
	GROUP BY department_id
) ag_dep
INNER JOIN job_grades g
ON ag_dep.ag BETWEEN lowest_sal AND highest_sal;
```

```mysql
#四、exists后面（相关子查询）
/*
语法：
exists(完整的查询语句)
结果：
1或0
*/
SELECT EXISTS(SELECT employee_id FROM employees WHERE salary=300000);

#案例1：查询有员工的部门名

#in
SELECT department_name
FROM departments d
WHERE d.`department_id` IN(
	SELECT department_id
	FROM employees
);

#exists
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT *
	FROM employees e
	WHERE d.`department_id`=e.`department_id`
);

#案例2：查询没有女朋友的男神信息

#in
SELECT bo.*
FROM boys bo
WHERE bo.id NOT IN(
	SELECT boyfriend_id
	FROM beauty
);

#exists
SELECT bo.*
FROM boys bo
WHERE NOT EXISTS(
	SELECT boyfriend_id
	FROM beauty b
	WHERE bo.`id`=b.`boyfriend_id`
);
```

### 分页查询

当待查询数据较多，需要分页显示时，可以使用分页查询，查询出部分数据，减小IO

```mysql
select 查询列表
from 表
[join type] join 表2
on 连接条件
where 筛选条件
group by 分组字段
having 分组后的筛选
order by 排序的字段】
limit [offset], size;
```

注意
offset要显示条目的起始索引（起始索引从0开始）
size 要显示的条目个数`
规律：limit (page-1)*size,size;

```mysql
#案例1：查询前五条员工信息
SELECT * FROM  employees LIMIT 0,5;
SELECT * FROM  employees LIMIT 5;

#案例2：查询第11条——第25条
SELECT * FROM employees LIMIT 10,15;

#案例3：有奖金的员工信息，并且工资较高的前10名显示出来
SELECT * FROM employees 
WHERE commission_pct IS NOT NULL 
ORDER BY salary DESC LIMIT 10 ;
```

### 联合查询

union (联合、合并)：将多条查询语句的结果合并成一个结果。

作用：

1. 将一条比较复杂的查询语句拆分成多条语句
2. 适用于查询多个表的时候，查询的列基本是一致。

特点：

1. 要求多条查询语句的查询列数是一致的！
2. 要求多条查询语句的查询的每一列的类型和顺序最好一致
3. union关键字默认去重，如果使用union all 可以包含重复项

```mysql
SELECT id,cname,csex FROM t_ca WHERE csex='男'
UNION
SELECT t_id,tName,tGender FROM t_ua WHERE tGender='male';
```

