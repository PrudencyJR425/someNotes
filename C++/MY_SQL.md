![image-20230813122756135](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230813122756135.png)

![image-20230813123117302](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230813123117302.png)





## MySQL介绍

![image-20230814114826004](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814114826004.png)

![image-20230814114924489](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814114924489.png)

#### RDBMS 关系型数据库

![image-20230814114719184](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814114719184.png)



#### 非RDBMS

键值型--Redis.



### 关系型数据库设计规则

ORM思想:对象关系映射

数据库一个表<------>一个类

表中一条数据<---------->类的一个对象

表中一个列<--------->类的一个属性

![image-20230814120225256](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814120225256.png)



### 表的关联关系

#### 一对一关联

![image-20230814120618153](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814120618153.png)

### 

#### 一对多关系

![image-20230814120724281](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814120724281.png)



#### 多对多关系

![image-20230814120737980](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814120737980.png)

![image-20230814120756238](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814120756238.png)





### MySQL使用

`mysql -uroot -p密码  //-P端口`

==1.查看数据库==

`show databases;`

![image-20230814152515039](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814152515039.png)

==2.创建数据库==

`create database atguigudb;`

==3.使用自己的数据库==

`use atguigudb;`

使用完use语句之后，如果接下来的SQL都是针对一个数据库操作的，那就不用重复use了，如果要针对另
一个数据库操作，那么要重新use。  

==4.查看某个库的所有表格==

`show tables from 库名;`

==5.创建新的表格==

```
create table 表名称 (字段名 数据类型, 字段名 数据类型);
```

e.g.

```
create table student(id int,name varchar(20));
```

==6.查看一个表格内容==

`select* from 数据库表名称;`

==7.添加一条记录==

`insert into 表名称 values(值列表);`

e.g.

`insert into student values(1,'ZhangSan');`

==8.查看表的创建信息==

`show create table 表名称\G`

==9.查看数据库创建信息==

`show create database 数据库名\G`

//P.S.   字符集latin1,不支持中文

![image-20230814154133344](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814154133344.png)

这里字符集是utf8.支持中文。

==10. 删除表格==

`drop table 表名称;`

==11.删除数据库==

`drop database 数据库名;`



### MySQL目录结构

![image-20230814163451742](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814163451742.png)





## C3 SQL_SELECT

```sql
# 基本的select语言
/*
DDL：数据定义语言; CREATE\ALTER\DROP\RENAME\TRUNCATE(清空)

DML: 数据操作语言; INSERT\DELETE\UPDATE\SELECT		(对数据增删改查)

DCL: 数据控制语言。COMMIT\ROLLBACK(事务) \SAVEPOINT\GRANT(赋权)\ REVOKE(回收权限)

*/

#数据导入指令
#1#命令行下:
mysql> source d:\mysqldb.sql;
可视化工具Navicat下:右击localhost->运行sql文件
#2#select语句结构: select 字段1,字段2..from 表名;      select * 表示所有字段(列)
#3#列的别名
select employee_id as emp_id,last_name from employee;			#emp_id就是别名
#4#去除重复行 select distinct department_id;
#5#空值:null 参与运算不等同于0，Null参与运算结果一定是null。这里别名" "
select employee_id,salary "月工资",salary*(1+IFNULL(commission_pct,0))*12 "年工资",commission_pct from employees;		#commission_pct为null时，如果不加IFNULL,那么年工资也为null。
#6#着重号 '';字段名字与关键字冲突；要使用字段名要使用着重号
select * from 'order';
#7#查询常数,将给每一行都分配一个常数。
select '尚硅谷',employee_id,last_name from employees;	
#8#显示表结构;显示字段详细信息。
decribe employees;
#9#过滤数据;查询90号部门员工信息
select * from employees where department_id = 90;
```

练习:

1.查询员工12个月的工资总和，并起别名为ANNUAL SALARY  

`select employee_id, last_name, salary*12 "ANNUAL SALARY" from employees;`

2.查询employees表中去除重复的job_id以后的数据  

`select distinct job_id from employees;`

3.查询工资大于12000的员工姓名和工资  

```sql
select employee_id,last_name,salary from employees where salary>12000;
```

4.查询员工号为176的员工的姓名和部门号  

```sql
select department_id,last_name from employees where employee_id = 176
```



## C4 运算符

### 算数运算符

from dual 伪表

```sql
select 100,100+0,100-0,100+50,100+35.5,100-35.5 from dual;
select 12 % 3,12 mod 5 from dual;
select * from employees where employee_id mod 2 = 0;
```

### 比较运算符

==等于运算符是===

如果等号两边的值、字符串或表达式都为字符串，则MySQL会按照字符串进行比较，其比较的
是每个字符串中字符的ANSI编码是否相等。
如果等号两边的值都是整数，则MySQL会按照整数来比较两个值的大小。
==如果等号两边的值一个是整数，另一个是字符串，则MySQL会将字符串转化为数字进行比较==。
==如果等号两边的值、字符串或表达式中有一个为NULL，则比较结果为NULL==

![image-20230814182238526](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230814182238526.png)

使用安全等于运算符时，两边的操作数的值都为NULL时，返回的结果为1而不是NULL，其他
返回结果与等于运算符相同  



### 空运算符

判断一个值是否为NULL。为NULL则返回1。否则0

```sql
select ISNULL(NULL);
```



### 最值运算符

当参数是整数或者浮点数时，LEAST将返回其中最小的值；

当参数为字符串时，返回字母表中顺序最靠前的字符；

当比较值列表中有NULL时，不能判断大小，返回值为NULL。  

```
least();
greatest();
```



### BETWEEN_AND 

```sql
select last_name,salary from employees where salary between 2400 and 3000;
```

### IN/NOT IN运算符

当参数是整数或者浮点数时，LEAST将返回其中最小的值；

当参数为字符串时，返回字母表中顺序最靠前的字符；

==当比较值列表中有NULL时，不能判断大小，返回值为NULL==。  

```sql
select last_name from employees where manager_id in(100,101,201);
```



### LIKE运算符

LIKE运算符主要用来匹配字符串，通常用于模糊匹配，如果满足条件则返回1，否则返回0。  

如果给定的值或者匹配条件为NULL，则返回结果为NULL。  

```
“%”：匹配0个或多个字符。
“_”：只能匹配一个字符。(占位)
```

```sql
select NULL like 'abc','abc' like NULL;
```

```sql
#查询第2个字符是_且第3个字符是'a'的员工信息：使用转义字符\
select last_name from employees where last_name like '_\_a';
select last_name from employees where last_name like 'S%';
```



### REGEXP 正则表达式

精细地匹配

C4扩展



### 逻辑运算符

NOT、AND、OR、XOR

逻辑异或（XOR）运算符是当给定的值中任意一个值为NULL时，则返回NULL；

OR当一个值为NULL，并且另一个值为非0值时，返回1，否则返回NULL；  



练习

```sql
1.选择工资不在5000到12000的员工的姓名和工资
select last_name,salary from employees where salary not between 5000 and 12000; 
2.选择在20或50号部门工作的员工姓名和部门号
select department_id,last_name from employees where department_id between 20 and 50;
3.选择公司中没有管理者的员工姓名及job_id
select last_name,job_id from employees where ISNULL(manager_id);
4.选择公司中有奖金的员工姓名，工资和奖金级别
select last_name,salary from employees where not ISNULL(commission_pct);
5.选员工姓名的第三个字母是a的员工姓名
select last_name from employees where last_name like '__a%';
6.选择姓名中有字母a和k的员工姓名
select last_name from employees where last_name like 'a%k%' or last_name like 'k%a%';
7.显示出表 employees 表中 first_name 以 'e'结尾的员工信息
select employee_id from employees where first_name like '%e';
8.显示出表 employees 部门编号在 80-100 之间的姓名、工种
select employee_id ,job_id from employees where department_id in(80,100);
```



## C5 排序与分页

默认查询按照数据添加顺序显示。

#按照salary从高到低顺序显示员工信息。 ORDER BY

#升序: ASC    (ascend)

#降序: DESC (descend)

```sql
select employee_id,last_name,salary from employees order by salary DESC;	//默认ASC
```

#二级排序

#显示员工信息，按照department_id降序排，salary升序排

```sql
select employee_id,last_name,salary from employees order bt department_id DESC,salary ASC;
```



#分页 limit <偏移量><条目数>

数据太多，分页处理

#偏移量0，显示20条记录。

```sql
select employee_id,last_name from employees limit 0,20;
```

#显示第2页。

```sql
select employee_id,last_name from employees limit 20,20;
```



#练习

```sql
#1. 查询员工的姓名和部门号和年薪，按年薪降序,按姓名升序显示
select last_name,salary*12 from employees order by salary DESC,last_name ASC;
#2. 选择工资不在 8000 到 17000 的员工的姓名和工资，按工资降序，显示第21到40位置的数据
select last_name,salary from employees where salary not between 8000 and 17000 order by salary DESC limit 21,20;

#3. 查询邮箱中包含 e 的员工信息，并先按邮箱的字节数降序，再按部门号升序
select last_name,email,department_id from employees where email regexp '[e]' order by
length(email) DESC,department_id ASC;
```





## C6 多表查询

![image-20230815101337528](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230815101337528.png)

```sql
DESC employees;
DESC departments;
DESC locations;

//查询员工名为'Abel'的人在哪个城市工作？
select* from employees where last_name = 'Abel';
select* from departments where department_id = 80;
select* from locations where location_id = 2500;
```

不分表格将会产生大量冗余；效率太差。



#多表查询实现

### 笛卡尔积错误

（交叉连接）

```sql
#错误实现:每个员工与每个部门都匹配了；笛卡尔积错误。
select employee_id,department_name from employees,departments; #2889条记录 107员工*27部门

#正确方式：需要连接条件   n个表至少需要n-1个连接条件
#两个表的连接
select employee_id,department_name from employees,departments where employees.department_id = departments.department_id;
#建议多表查询时，每个字段前都指明其所在的表；
select employees.employee_id,departments.department_name,employees.department_id from employees,departments where employees.department_id = departments.department_id;

#可以给表起别名，在select和where中使用别名。 先执行from->where->select
select t1.employee_id,t2.department_name,t1.department_id from employees t1,departments t2 where t1.department_id = t2.department_id;

#练习：查询员工的employee_id,last_name,department_name,city
select employee_id,last_name,department_name,city
from employees,departments,locations
where employees.department_id = departments.department_id and
	  departments.location_id = locations.location_id;
```



### 多表查询分类

1.等值 vs 非等值

```sql
//非等值
select last_name,salary,grade_level
from employees e,job_grades j
where e.salary >= j.lowest_sal and e.salary <= j.highest_sal;
```

2.自连接 vs 非自连接

```sql
//自连接
//查询员工姓名及其管理者的id和姓名。
select emp.employee_id,emp.last_name,mgr.employee_id,mgr.last_name 
from employees emp,employees mgr
where emp.manager_id = mgr.employee_id;
```

3.内连接 vs 外连接

```sql
//内连接	取匹配行。合并----交集
select employee_id,department_name
from employees e,departments d
where e.department_id = d.department_id;

//外连接 join...on(内连接也可以用join..on)
//外连接分类:左外连接//右外连接//满外连接----VN图
//查询员工*所有*last_name,department_name信息.左外连接
select employee_id,department_name
from employees e left outer join departments d
on e.department_id = d.department_id;

//右外连接
select employee_id,department_name
from employees e right outer join departments d
on e.department_id = d.department_id;
```

#### union

![image-20230815111551997](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230815111551997.png)

#### SQL_JOINS

![image-20230815111737691](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230815111737691.png)

```sql
#中:内连接
select employee_id,department_name
from employees e join departments d
on e.department_id = d.department_id;
#左上:左外连接
select employee_id,department_name
from employees e left join departments d
on e.department_id = d.department_id;
#右上:右外连接
select employee_id,department_name
from employees e right join departments d
on e.department_id = d.department_id;
#左中：
select employee_id,department_name
from employees e left join departments d
on e.department_id = d.department_id
where isnull(d.department_id);
#右中:
select employee_id,department_name
from employees e right join departments d
on e.department_id = d.department_id
where isnull(e.department_id);
#左下:左上 union all 右中
select employee_id,department_name
from employees e left join departments d
on e.department_id = d.department_id
union all
select employee_id,department_name
from employees e right join departments d
on e.department_id = d.department_id
where isnull(e.department_id);
```



练习

```sql
# 1.显示所有员工的姓名，部门号和部门名称。
select e.last_name,e.department_id,d.department_name
from employees e left join departments d
on e.department_id = d.department_id;

# 2.查询90号部门员工的job_id和90号部门的location_id
select e.job_id,l.location_id
from employees e,jobs j,locations l,departments d
where e.job_id = j.job_id and d.department_id = 90 and d.location_id = l.location_id;

# 3.选择所有有奖金的员工的 last_name , department_name , location_id , city
select e.last_name,d.department_name,l.location_id,l.city
from employees e left join departments d
on e.department_id = d.department_id
left join locations l
on d.location_id = l.location_id
where not isnull(e.commission_pct);
```



## C7 单行函数

连接字符串_ concat

```sql
select concat("hello","world") from dual;
```

![image-20230817113500675](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817113500675.png)

### 基本函数

![image-20230817113535005](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817113535005.png)

```sql
select ABS(-124),SIGN(-1),PI(),CEIL(32.32),FLOOR(12.3),MOD(12,5) from dual;
```



### 三角函数

#### 角度弧度互换

```
RADIANS(x);								//角度-->弧度
DEGREES(x);								//弧度-->角度
```

![image-20230817114412775](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817114412775.png)



### 指数与对数

![image-20230817114911393](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817114911393.png)



### 进制转换

![image-20230817115048561](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817115048561.png)



### 字符串函数

![image-20230817115427073](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817115427073.png)

![image-20230817115543105](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817115543105.png)

![image-20230817115630684](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817115630684.png)

(MySQL字符串位置从1开始。)

```
#LPAD:实现右对齐
#RPAD:实现左对齐
select employee_id,last_name,LPAD(salary,10,' ') from employees;

```







### 日期和时间函数

#### 获取日期和时间

![image-20230817145631076](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817145631076.png)

```sql
SELECT CURDATE(),CURRENT_DATE(),CURTIME(),NOW(),SYSDATE(),UTC_DATE(),UTC_TIME()
from dual;
```



#### 时间戳

![image-20230817145713475](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817145713475.png)

```sql
SELECT UNIX_TIMESTAMP(),UNIX_TIMESTAMP('2021-10-01 12:12:32'),
FROM_UNIXTIME(1692256229),FROM_UNIXTIME(1633061552)
from dual;
```



#### 获取月份、星期、星期数、天数函数

![image-20230817151338893](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817151338893.png)

#### 日期操作函数

![image-20230817151914621](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817151914621.png)

```sql
select extract(second from now()),extract(quarter from '2021-5-12');
```





#### 时间-秒转换

![image-20230817152113047](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817152113047.png)

```sql
select time_to_sec(now());
```





#### 计算日期与时间的函数

![image-20230817152214473](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817152214473.png)

```sql
select now(),date_add(now(),interval 1 year) from fual;
```

![image-20230817152626769](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817152626769.png)





#### 日期格式化与解析

格式化: 日期 --- > 字符串

解析   ：字符串 ---> 日期

![image-20230817153517604](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817153517604.png)

![image-20230817153931330](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817153931330.png)

```sql
#格式化
select DATE_FORMAT(CURDATE(),'%Y-%M-%D'),TIME_FORMAT(curtime(),'%h:%i:%S'),
DATE_FORMAT(NOW(),'%Y-%M-%D %h:%i:%s %a %T %p')
from dual;

#解析
select STR_TO_DATE('2023-August-17th 03:42:30','%Y-%M-%D %h:%i:%s')
from dual;
****
select get_format(DATE,'USA') from dual;
select date_format(curdate(),get_format(DATE,'USA')) from dual;
```



### 流程控制函数

![image-20230817154858613](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817154858613.png)

```sql
#if(value,value1,value2)
select last_name,salary,if(salary >= 6000,'高工资','低工资') "details"
from employees;
select last_name,commission_pct,if(commission_pct is not null,commission_pct,0) "details"
from employees;

#case when ... then ... when ... then .. else .. end
select last_name,salary,case when salary >= 15000 then '白骨精'
							 when salary >= 10000 then '潜力股'
							 when salary >= 8000 then '小屌丝'
							 else '草根' end "details",department_id
from employees;

#case
select employee_id,last_name,department_id,salary,
							 case department_id when 10 then salary*1.1
							 					when 20 then salary*1.2
							 					when 30 then salary*1.3
							 					else salary*1.4 end "details"
from employees;

#只显示department_id 在10，20，30的。
select employee_id,last_name,department_id,salary,
							 case department_id when 10 then salary*1.1
							 					when 20 then salary*1.2
							 					when 30 then salary*1.3
							 					end "details"
from employees
where department_id in (10,20,30);
```





### 加密解密函数

![image-20230817161012415](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817161012415.png)

```sql
#password已经弃用。 encode/decode 已经弃用。
#MD5，sha
select MD5('mysql'),sha('mysql')
from dual
```

### MySQL信息函数

![image-20230817161652606](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230817161652606.png)



```sql
# 1.显示系统时间(注：日期+时间)
select curdate(),curtime() from dual;
select now() from dual;
# 2.查询员工号，姓名，工资，以及工资提高百分之20%后的结果（new salary）
select last_name,salary,salary*1.2 "new salary" from employees;
# 3.将员工的姓名按首字母排序，并写出姓名的长度（length）----------------
select last_name,length(last_name) from employees order by last_name desc;
# 4.查询员工id,last_name,salary，并作为一个列输出，别名为OUT_PUT
select concat(employee_id,"-",last_name,"-",salary) from employees;
# 5.查询公司各员工工作的年数、工作的天数，并按工作年数的降序排序-----------
select DATEDIFF(sysdate(),hire_date) / 365 work_years,DATEDIFF(sysdate(),hire_date) work_days 
from employees
order by work_years desc;
# 6.查询员工姓名，hire_date , department_id，满足以下条件：雇用时间在1997年之后，department_id为80 或90或110, commission_pct不为空
select last_name,hire_date,department_id
from employees
where hire_date > 1997 and department_id in (80,90,110) and  commission_pct is not null;
# 9.使用case-when，按照下面的条件
# case-when 没有逗号，then是给别名grade赋值。
select last_name "Last_name",job_id "Job_id",case job_id
					when 'AD_PRES' then 'A'
				    when 'ST_MAN' then 'B'
				    when 'IT_PROG' then 'C'
				    when 'SA_REP' then 'D'
				    when 'ST_CLERK' then 'E'
				    else 'F' end "grade"
from employees;
```





## C8 常用聚合函数

### AVG/SUM

```sql
select AVG(salary),SUM(salary) from employees;
#只适合数值类型，不计算空值。
```

```sql
#查询平均奖金率
select AVG(commission_pct) from employees;
###！！！错误！！！AVG忽略NULL值，没算没有奖金的员工。
#修改
select AVG(IFNULL(commission_pct,0)) from employees;	//NULL按照0处理
```



### MAX/MIN

```sql
select max(salary),min(salary) from employees;
#可以填入字符串、日期时间类型。
```

### COUNT

计算指定字段在查询结构中出现的个数 (行数)

查询表中多少条记录: count(1)//count(*)

count计算指定字段个数时，不计算NULL值。

```sql
select COUNT(employee_id),COUNT(salary),count(1) from employees;
#count(1) 每一行算1累计。
```

==P.S.count(1) 与 count(*)效率==

MyISAM引擎下三者效率一样。

InnoDB引擎下：count(*)  = count(1) > count(字段)； 不需要找二级索引了，更快。



### GROUP BY

分组

```sql
#求员工表各个部门平均工资
select department_id,AVG(salary) from employees GROUP BY department_id;

##多列分组：不同部门，不同工种分别分组
select department_id,job_id,AVG(salary) from employees group by department_id,job_id;

### with rollup 还做一次整体平均。 和order by一起用时需要注意。
```

### HAVING

1.过滤数据:如果过滤条件中使用了聚合函数MAX...==必须使用Having替换Where==。

2.HAVING必须声明在GROUP_BY之后。

3.当过滤条件没有聚合函数，则此条件声明在where中更快。

```sql
#查询各个部门中最高工资比10000高的部门信息
select department_id,MAX(salary) 
from employees Group by department_id HAVING MAX(salary) > 10000;

#查询部门id为10，20，30，40这四个部门中最高工资比10000高的部门信息。
select department_id,MAX(salary)
from employees 
GROUP BY department_id HAVING MAX(salary) > 10000 and department_id in(20,30,40,50);
#or 效率更高
select department_id,MAX(salary)
from employees 
where department_id in(20,30,40,50)
GROUP BY department_id HAVING MAX(salary) > 10000 
```

#### ==where&having==对比

HAVING可以完成WHERE不能完成的任务：聚合函数

但是WHERE对非聚合函数执行效率要高于HAVING。



### SQL底层原理

#### select语法

SELECT ...,...,...

FROM ...,...,...(LEFT/RIGHT) JOIN ... ON 多表连接条件

WHERE AND 不包含聚合条件

GROUP BY ...

HAVING 聚合条件

ORDER BY ...,... (ASC/DESC)

LIMIT ...,...



#### SQL语句执行顺序

![image-20230821222225000](C:\Users\张逸\AppData\Roaming\Typora\typora-user-images\image-20230821222225000.png)



练习

```sql
#1.where子句可否使用组函数进行过滤?
where不可以使用聚合函数进行过滤。
#2.查询公司员工工资的最大值，最小值，平均值，总和
SELECT MAX(salary),MIN(salary),AVG(salary),SUM(salary) from employees;
#3.查询各job_id的员工工资的最大值，最小值，平均值，总和
SELECT MAX(salary),MIN(salary),AVG(salary),SUM(salary) from employees GROUP BY job_id;
#4.选择具有各个job_id的员工人数
SELECT job_id,COUNT(*) from employees GROUP BY job_id;
#5.查询员工最高工资和最低工资的差距（DIFFERENCE）
SELECT (MAX(salary)-MIN(salary)) "DIF" from employees;
# 6.查询各个管理者手下员工的最低工资，其中最低工资不能低于6000，没有管理者的员工不计算在内
SELECT manager_id,MIN(salary) from employees where manager_id IS NOT NULL
GROUP BY manager_id HAVING MIN(salary)>6000;            
#7.查询所有部门的名字，location_id，员工数量和平均工资，并按平均工资降序
SELECT department_name,location_id,count(employee_id),AVG(salary) "avg_sal"
FROM employees e RIGHT JOIN departments d
ON e.department_id = d.department_id
GROUP BY department_name,location_id
ORDER BY avg_sal DESC;
```

