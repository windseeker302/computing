# mysql

## 简介

### 概述

mysql 其实是一个 `DBMS` 软件系统， 也就是说 mysql 并不是一款数据库，这是很多人的误区，我们经常听到的 oracle，mysql，微软SQL server都是DBMS软件系统，我们只是通过这些系统来管理和维护数据库而已，DBMS 相当于用户和数据库之间的桥梁，目前有超过300种不同的 DBMS 系统，难道我们都要学吗？还好数据库是有分类的，我们今天要了解的 mysql 对应的是关系型数据库，关系型数据库的数据存储模型很像 Excel，用行和列组织数据，操作关系型数据库的DBMS系统，大多数都是使用 SQL 来管理数据，因此学会SQL就相当于学会了这些 DBMS 的核心，DBMS 是什么呢，其实就是一款编程语言，不要觉得很难，这么多编程语言里，SQL的入门算是简单级别的了，那么大家就知道学习的核心是SQL。

组成：

- 数据库（database,DB），数据存储的仓库，数据是有组织的进行存储。

- 数据库操作系统（Database Management System,DBMS），操纵和管理数据库的大型软件。

- SQL（Structured Query Language），操纵关系型数据库的编程语言，定义了一套操作关系型数据库统一标准。


主流的关系型数据库管理系统：

- Oracle：大型收费。
- MySql：开源免费的中小型数据库。
- SQLServer(Microsoft SQL Server)：微软公司中型数据库，收费。
- PostgreSQl：开源免费的中小型数据库。
- SQLite：嵌入式微型数据库，在安卓内置的数据库中，就用的SQLite。

 

### 安装

#### 版本

MySQL 官方提供了两种不同的版本：

- 社区版（Community）

  免费，mysql不提供任何技术支持。

- 商业版（Enterprise Edition）

  收费，试用30天，官方提供技术支持。

#### 下载

下载地址：[MySQL :: MySQL Community Downloads](https://dev.mysql.com/downloads/)

## SQL

### SQL通用语法

1. SQL语句可以单行或多行书写，以分号结尾。
2. SQL语句可以使用空格/缩进来增强语句的可读性。
3. MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。
4. 注释：

   - 单行注释：--注释内容或#注释内容(MySQL特有) 

   - 多行注释：/\*注释内容*/


### SQL分类

DDL（Data Definition Language）：数据定义语言，用来定义数据库对象（数据库，表，字段）。

DML（Data Manipulation Language）：数据操作语言，用来对数据库中的数据进行增删改。

DQL（Data Query Language）：数据查询语言，用来查询数据库中表的记录。

DCL（Data Control Language）：数据控制语言，用来控制数据库用户，控制数据库的访问权限。

### DDL

#### 数据库操作

- 查询

  查询所有数据库

  ~~~sql
  show DATABASES;
  ~~~

  

  查询当前数据库

  ~~~sql
  select DATABASE();
  ~~~

  

- 创建

  ~~~sql
  CREATE DATABASE [IF NOT EXISTS] 数据库名称 [DEFAULT CHARSET 字符集] [COLLATE 排列顺序];
  ~~~

  

- 删除

  ~~~sql
  DROP DATABASE [IF EXISTS] 数据库名称;
  ~~~

  

- 使用

  ~~~sql
  USE 数据库名称;
  ~~~



#### 表操作

##### 查询

- 查询当前所有表

  ~~~sql
  SHOW TABLES;
  ~~~

  

- 查询表结构

  ~~~sql
  DESC 表名;
  ~~~

  

- 查询指定表的建表语句

  ~~~sql
  SHOW CREATE TABLE 表名;
  ~~~



##### 创建

- 创建

  ~~~sql
  CREATE TABLE 表名(
  	字段1 字段1类型 [COMMENT 字段1注释],
      字段2 字段2类型 [COMMENT 字段2注释],
      字段3 字段3类型 [COMMENT 字段3注释],
      ......
      字段n 字段n类型 [COMMENT 字段n注释],
  )[COMMENT 表注释];
  ~~~



- 数据类型

  MySQL中的数据类型有很多，主要分为三类：数值类型、字符串类型、日期时间类型。

  数值类型：

  | 类型         | 大小                                     | 范围（有符号）                                               | 范围（无符号）                                               | 用途           |
  | :----------- | :--------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :------------- |
  | TINYINT      | 1 Bytes                                  | (-128，127)                                                  | (0，255)                                                     | 小整数值       |
  | SMALLINT     | 2 Bytes                                  | (-32 768，32 767)                                            | (0，65 535)                                                  | 大整数值       |
  | MEDIUMINT    | 3 Bytes                                  | (-8 388 608，8 388 607)                                      | (0，16 777 215)                                              | 大整数值       |
  | INT或INTEGER | 4 Bytes                                  | (-2 147 483 648，2 147 483 647)                              | (0，4 294 967 295)                                           | 大整数值       |
  | BIGINT       | 8 Bytes                                  | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807)      | (0，18 446 744 073 709 551 615)                              | 极大整数值     |
  | FLOAT        | 4 Bytes                                  | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38)                  | 单精度浮点数值 |
  | DOUBLE       | 8 Bytes                                  | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 双精度浮点数值 |
  | DECIMAL      | 对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2 | 依赖于M和D的值                                               | 依赖于M和D的值                                               | 小数值         |

  字符串类型：

  | 类型       | 大小                  | 用途                            |
  | :--------- | :-------------------- | :------------------------------ |
  | CHAR       | 0-255 bytes           | 定长字符串                      |
  | VARCHAR    | 0-65535 bytes         | 变长字符串                      |
  | TINYBLOB   | 0-255 bytes           | 不超过 255 个字符的二进制字符串 |
  | TINYTEXT   | 0-255 bytes           | 短文本字符串                    |
  | BLOB       | 0-65 535 bytes        | 二进制形式的长文本数据          |
  | TEXT       | 0-65 535 bytes        | 长文本数据                      |
  | MEDIUMBLOB | 0-16 777 215 bytes    | 二进制形式的中等长度文本数据    |
  | MEDIUMTEXT | 0-16 777 215 bytes    | 中等长度文本数据                |
  | LONGBLOB   | 0-4 294 967 295 bytes | 二进制形式的极大文本数据        |
  | LONGTEXT   | 0-4 294 967 295 bytes | 极大文本数据                    |

  日期时间类型：

  | DATE      | 3    | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
  | --------- | ---- | ------------------------------------------------------------ | ------------------- | ------------------------ |
  | TIME      | 3    | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
  | YEAR      | 1    | 1901/2155                                                    | YYYY                | 年份值                   |
  | DATETIME  | 8    | '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'               | YYYY-MM-DD hh:mm:ss | 混合日期和时间值         |
  | TIMESTAMP | 4    | '1970-01-01 00:00:01' UTC 到 '2038-01-19 03:14:07' UTC结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYY-MM-DD hh:mm:ss | 混合日期和时间值，时间戳 |

##### 修改

- 添加字段

  ~~~sql
  ALTER TABLE 表名 ADD 字段名 类型(长度) [comment 注释] [约束] [after 字段名];
  ~~~

  

- 修改数据类型

  ~~~sql
  ALTER TABLE 表名 MODIFY 字段名 新数据类型(长度);
  ~~~

  

- 修改字段名和字段类型

  ~~~sql
  ALTER TABLE 表名 CHANGE 旧字段名 新字段名 类型(长度) [comment 注释] [约束];
  ~~~

  

- 删除字段

  ~~~sql
  ALTER TABLE 表名 DROP 字段名;
  ~~~



- 修改表名

  ~~~sql
  ALTER TABLE 表名 RENAME to 新表名;
  ~~~



##### 删除

- 删除表

  ~~~sql
  DROP TABLE [IF NOT EXISTS] 表名;
  ~~~

  

- 删除指定表，并重新创建该表

  ~~~sql
  TRUNCATE TABLE 表名;
  ~~~

  

### DML

#### 添加数据（INSERT）

- 给指定字段添加数据

  ~~~sql
  INSERT INTO 表名(字段名1,字段名2,...) VALUES(值1,值2,...);
  ~~~

  

- 给全部字段添加数据

  ~~~sql
  INSERT INTO 表名 VALUES(值1,值2,...);
  ~~~

  

- 批量添加数据

  ~~~sql
  # 指定字段添加数据
  INSERT INTO 表名(字段名1,字段名2,...) VALUES(值1,值2,...),(值1,值2,...),(值1,值2,...);
  # 全部字段添加数据
  INSERT INTO 表名 VALUES(值1,值2,...),(值1,值2,...),(值1,值2,...);
  ~~~

  

> 注意：
>
> - 插入数据时，指定的字段顺序需要与值的顺序是一一对应的。
> - 字符串和日期型数据应该包含在引号中。
> - 插入的数据大小，应该在字段的规定范围内。

#### 修改数据（UPDATE）

~~~sql
UPDATE 表名 SET 字段名1=值1,字段名2=值2,... [WHERE 条件];
~~~



> 注意：修改语句的条件可以有，也可以没有，如果没有条件，则会修改整张表的所有数据。



#### 删除数据（DELETE）

~~~sql
DELETE FROM 表名 [WHERE 条件];
~~~



> 注意：
>
> - DELETE语句的条件可以有，也可以没有，如果没有条件，则会删除整张表的所有数据。 
> - DELETE语句不能删除某一个字段的值(可以使用UPDATE)。

### DQL

语法

~~~sql
SELECT
	字段列表
FROM
	表名列表
WHERE
	条件列表
GROUP BY
	分组列表
HAVING
	分组条件
ORDER BY
	排序字段
LIMIT
	分页参数
~~~



#### 基础查询

- 查询多个字段

  ~~~sql
  SELECT 字段1,字段2,... FROM 表名;
  ~~~

  

- 查询返回所有字段

  ~~~sql
  SELECT * FROM 表名;
  ~~~

  

- 设置别名

  ~~~sql
  SELECT 字段1 [AS 别名1],字段2 [AS 别名2],... FROM 表名;
  ~~~



- 去除重复记录

  ~~~sql
  SELECT DISTINCT 字段列表 FROM 表名;
  ~~~

  

#### 条件查询（WHERE）

1. 语法：

   ~~~sql
   SELECT 字段列表 FROM 表名 WHERE 列表条件;
   ~~~

   - 基本查询

     

   - 模糊查询

     ~~~sql
     模糊查询
     关键字是like，通常位于条件字段后面
     
     语法：select 字段 from 表名 where 字段 like ‘%数据%’；
     ~~~

     

   

2. 条件

   A=10 B=20

   | 操作符 | 描述                                                         | 实例                 |
   | :----- | :----------------------------------------------------------- | :------------------- |
   | =      | 等号，检测两个值是否相等，如果相等返回true                   | (A = B) 返回false。  |
   | <>, != | 不等于，检测两个值是否相等，如果不相等返回true               | (A != B) 返回 true。 |
   | >      | 大于号，检测左边的值是否大于右边的值, 如果左边的值大于右边的值返回true | (A > B) 返回false。  |
   | <      | 小于号，检测左边的值是否小于右边的值, 如果左边的值小于右边的值返回true | (A < B) 返回 true。  |
   | >=     | 大于等于号，检测左边的值是否大于或等于右边的值, 如果左边的值大于或等于右边的值返回true | (A >= B) 返回false。 |
   | <=     | 小于等于号，检测左边的值是否小于或等于右边的值, 如果左边的值小于或等于右边的值返回true | (A <= B) 返回 true。 |

简单实例 

1. 等于条件：

```sql
SELECT * FROM users WHERE username = 'test';
```

2. 不等于条件：

```sql
SELECT * FROM users WHERE username != 'runoob';
```

3. 大于条件:

```sql
SELECT * FROM products WHERE price > 50.00;
```

4. 小于条件:

```sql
SELECT * FROM orders WHERE order_date < '2023-01-01';
```

5. 大于等于条件:

```sql
SELECT * FROM employees WHERE salary >= 50000;
```

6. 小于等于条件:

```sql
SELECT * FROM students WHERE age <= 21;
```

7. 组合条件（AND、OR）:

```sql
SELECT * FROM products WHERE category = 'Electronics' AND price > 100.00;

SELECT * FROM orders WHERE order_date >= '2023-01-01' OR total_amount > 1000.00;
```

8. 模糊匹配条件（LIKE）:

```sql
SELECT * FROM customers WHERE first_name LIKE 'J%';
```

9. IN 条件:

```sql
SELECT * FROM countries WHERE country_code IN ('US', 'CA', 'MX');
```

10. NOT 条件:

```sql
SELECT * FROM products WHERE NOT category = 'Clothing';
```

11. BETWEEN 条件:

```sql
SELECT * FROM orders WHERE order_date BETWEEN '2023-01-01' AND '2023-12-31';
```

12. IS NULL 条件

```sql
SELECT * FROM employees WHERE department IS NULL;
```

13. IS NOT NULL 条件:

```sql
SELECT * FROM customers WHERE email IS NOT NULL;
```

如果我们想在 MySQL 数据表中读取指定的数据，WHERE 子句是非常有用的。

使用主键来作为 WHERE 子句的条件查询是非常快速的。

如果给定的条件在表中没有任何匹配的记录，那么查询不会返回任何数据。



#### 聚合函数（count，max，min，avg，sum）

将一列数据作为一个整体，进行纵向计算。

常见聚合函数:

| 函数  | 功能     |
| ----- | -------- |
| count | 统计数量 |
| max   | 最大值   |
| min   | 最小值   |
| avg   | 平均值   |
| sum   | 求和     |

语法：

~~~sql
SELECT 聚合函数(字段列表) FROM 表名;
~~~



> 注意：null值不参与所有聚合函数运算。

#### 分组查询（GROUP BY）

语法：

~~~sql
SELECT 字段列表 FROM 表名 [WHERE 条件] GROUP BY 分组字段名 [HAVING 分组后过滤条件];
~~~

> where 和 having 区别？
>
> - 执行时机不同：where 是分组之前进行过滤，不满足 where 条件，不参与分组；而having 是分组后对结果进行过滤。
> - 判断条件不同：where 不能对聚合函数进行判断，而 having 可以。

示例：

1. 根据性别分组，统计男性员工数量和女性员工数量。

   ~~~sql
   select gender,count(*) from emp group by gender;
   ~~~

   

2. 根据性别分区，统计男性员工和女性员工的平均年龄。

   ~~~sql
   select gender,avg(age) from emp group by gender;
   ~~~

   

3. 查询年龄小于45的员工，并根据工作地址分组，获取员工数量大于等于3的工作地址。

   ~~~sql
   select * from emp where age < 45 group by workaddress;
   ~~~

使用 WITH ROLLUP

WITH ROLLUP 可以实现在分组统计数据基础上再进行相同的统计（SUM,AVG,COUNT…）。

例如我们将以上的数据表按名字进行分组，再统计每个人登录的次数：

```sql
mysql> SELECT name, SUM(signin) as signin_count FROM  employee_tbl GROUP BY name WITH ROLLUP;
+--------+--------------+
| name   | signin_count |
+--------+--------------+
| 小丽 |            2 |
| 小明 |            7 |
| 小王 |            7 |
| NULL   |           16 |
+--------+--------------+
4 rows in set (0.00 sec)
```

其中记录 NULL 表示所有人的登录次数。



#### 排序查询（ORDER BY）

语法：

~~~sql
SELECT 字段列表 FROM 表名 ORDER BY 字段1 排序方式1,字段2 排序方式2;
~~~



排序方式：

- ASC：升序（默认值）
- DESC：降序

> 注意：如果是多字段排序，当第一个字段相同，才会根据第二个字段排序。



#### 分页查询（LIMIT）

语法：

~~~sql
SELECT 字段列表 FROM 表名 LIMIT 起始索引,查询记录数;
~~~

> 注意：
>
> - 起始索引1从0开始，起始索引=（查询页码-1）* 每页显示记录数。
> - 分页查询是数据库的方言，不同的数据库有不同的实现，MySQL中是LIMIT。
> - 如果查询的是第一页数据，起始索引可以省略，直接简写为limit10。

#### 执行顺序

编写顺序：

~~~sql
SELECT
	字段列表
FROM
	表名列表
WHERE
	条件列表
GROUP BY
	分组字段列表
HAVING
	分组后条件列表
ORDER BY
	排序字段
LIMIT
	分页参数
~~~

执行顺序：

~~~sql
FROM
	表名列表
WHERE
	条件列表
GROUP BY
	分组字段列表
HAVING
	分组后条件列表
SELECT
	字段列表
ORDER BY
	排序字段
LIMIT
	分页参数
~~~



### DCL

#### 用户管理

- 查询用户

  ~~~sql
  USE mysql;
  SELECT user,host FROM user;
  ~~~

  

- 创建用户

  ~~~sql
  CREATE USER '用户名@主机名' IDENTIFIED BY '密码';
  ~~~

  

- 修改用户密码

  ~~~sql
  ALTER USER '用户名@主机名' IDENTIFIED WITH mysql_native_password BY '密码';
  ~~~

  

- 删除用户

  ~~~sql
  DROP USER '用户名@主机名';
  ~~~

  

#### 权限控制

- 查询权限

  ~~~sql
  SHOW GRANTS FOR '用户名@主机名';
  ~~~

  

- 授予权限

  ~~~sql
  GRANT 权限列表 ON 数据库名.表名 TO '用户名@主机名';
  ~~~

  

- 撤销权限

  ~~~sql
  REVOKE 权限列表 ON 数据库名.表名 FROM '用户名@主机名';
  ~~~

  

  > 注意:
  >
  > - 多个权限之间，使用逗号分隔。
  > - 授权时，数据库名和表名可以使用*进行通配，代表所有。

## 图形化管理系统（软件）

- ​	
  - 下载地址：https://sqlyog.en.softonic.com/

- Navicat
  - 下载地址：[Navicat | 下载 Navicat Premium 14 天免费 Windows、macOS 和 Linux 的试用版](https://navicat.com.cn/download/navicat-premium)

- DataGrip
  - 下载地址：https://www.jetbrains.com/datagrip/download/




## 函数

`函数 `是指一段可以直接被另一段程序调用的程序或代码。

使用函数：

~~~sql
SELECT 函数(参数);
~~~



### 字符串函数

- 常用字符串函数

  | 函数                     | 功能                                                      |
  | ------------------------ | --------------------------------------------------------- |
  | CONCAT(S1,S2,...Sn)      | 字符串拼接，将S1，S2，...Sn拼接成一个字符串               |
  | LOWER(str)               | 将字符串 str 全部转为小写                                 |
  | UPPER(str)               | 将字符串 str 全部转为大写                                 |
  | LPAD(str,n,pad)          | 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度 |
  | RPAD(str,n,pad)          | 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度 |
  | TRIM(str)                | 去掉字符串头和尾的空格                                    |
  | SUBSTRING(str,start,len) | 返回从字符串str从start位置起的len个长度的字符串           |

  

- 示例

  由于业务需求变更，企业员工的工号，统一为5位数，目前不足5位数的全部在前面补0。比如：1号员工的工号应该为00001。

  ~~~sql
  update tmp set workno = LPAD(workno,5,'0');
  ~~~

  

### 数值函数

- 常用数值函数：

  | 函数       | 功能                           |
  | ---------- | ------------------------------ |
  | CEIL(x)    | 向上取整                       |
  | FLOOR(x)   | 向下取整                       |
  | MOD(x,y)   | 返回x/y的模(取余)              |
  | RAND()     | 返回0~1的随机数                |
  | ROUND(x,y) | 求参数x的四舍五入，保留y位小数 |

  

- 示例：

  通过数据库的函数，生成一个六位数的随机验证码。

  ~~~sql
  select lpad(round(rand()*1000000,0),6,'0');
  ~~~

  

### 日期函数

- 常用的日期函数：

  | 函数                              | 功能                                              |
  | --------------------------------- | ------------------------------------------------- |
  | CURDATE()                         | 返回当前日期                                      |
  | CURTIME()                         | 返回当前时间                                      |
  | NOW()                             | 返回当前日期和时间                                |
  | YEAR(date)                        | 获取指定date的年份                                |
  | MONTH(date)                       | 获取指定date的月份                                |
  | DAY(date)                         | 获取指定date的日期                                |
  | DATE_ADD(date,INTERVAL expr type) | 返回一个日期/时间值加上一个时间间隔expr后的时间值 |
  | DATEDIFF(date1,date2)             | 返回起始时间date1和结束时间date2之间的天数        |

  

- 示例：

  查询所有员工的入职天数，并根据入职天数倒序排序。

  ~~~sql
  select name,datediff(curdate(),entrydate) AS 'entrydays' from emp order by entrydays desc;
  ~~~

  

### 流程函数

流程函数也是很常用的一类函数，可以在SQL语句中实现条件筛选，从而提高语句的效率。

常用的流程函数：

| 函数                                                       | 功能                                                     |
| ---------------------------------------------------------- | -------------------------------------------------------- |
| IF(value,t,f)                                              | 如果value位true，则返回t，否则返回f                      |
| IFNULL(value1,value2)                                      | 如果value1不为null，返回value1，否则返回value2           |
| CASE WHEN [val1] THEN [res1] ... ELSE [default] END        | 如果val1为true，返回resl，..否则返回default默认值        |
| CASE [expr] WHEN [val1] THEN [res1] ... ELSE [default] END | 如果expr的值等于val1，返回res1，...否则返回default默认值 |

示例：

统计班级各个学员的成绩，展示的规则如下：

- \>= 85，展示优秀 
- \>= 60，展示及格
- 否则，展示不及格

~~~sql
select 
	id,
	name,
	(case when math >= 85 then '优秀' when math >= 60 then '及格' else '不及格' end ) '数学',
	(case when english >= 85 then '优秀' when english >= 60 then '及格' else '不及格' end ) '英语',
	(case when chinese >= 85 then '优秀' when chinese >= 60 then '及格' else '不及格' end ) '语文',
from score;
~~~



## 约束

### 概述

约束是作用于表中字段上的规则，用于限制存储在表中的数据。

目的：保证数据库中数据的正确、有效性和完整性。

分类：

| 约束                     | 描述                                                     | 关键字         |
| ------------------------ | -------------------------------------------------------- | -------------- |
| 非空约束                 | 限制该字段的数据不能为null                               | NOT NULL       |
| 唯一约束                 | 保证该字段的所有数据都是唯一、不重复的                   | UNIQUE         |
| 主键约束                 | 主键是一行数据的唯一标识，要求非空且唯一                 | PRIMARY KEY    |
| 默认约束                 | 保存数据时，如果未指定该字段的值，则采用默认值           | DEFAULT        |
| 外键约束                 | 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 | FOREIGN KEY    |
| 检查约束(8.0.16版本之后) | 保证字段值满足某一个条件                                 | CHECK          |
| 自动增长                 | 从 1 2 3 ... n 依次递增                                  | AUTO_INCREMENT |



### 约束演示

| 字段名 | 字段含义   | 字段类型    | 约束条件               | 约束关键字                  |
| ------ | ---------- | ----------- | ---------------------- | --------------------------- |
| id     | ID唯一标识 | int         | 主键，并且自动增长     | PRIMARY KEY，AUTO_INCREMENT |
| name   | 姓名       | varchar(10) | 不为空，且唯一         | NOT NULL，UNIQUE            |
| age    | 年龄       | int         | 大于0，并且小于等于120 | CHECK                       |
| status | 状态       | char(1)     | 如果没有指定，默认为1  | DEFAULT                     |
| gender | 性别       | char(1)     | 无                     |                             |

~~~sql
create table user(
	id int primary key auto_increment comment 'ID',
    name varchar(10) not null unique comment '姓名',
    age int check ( age > 0 && age <= 120 ) comment '年龄',
    status char(1) default '1' comment '状态',
    gender char(1) comment '性别'
) comment '用户表';
~~~



### 外键约束

MySQL的外键约束用来在两个表数据之间建立链接，其中一张表的一个字段被另一张表中对应的字段约束。也就是说，设置外键约束至少要有两种表，被约束的表叫做从表（子表），另一张叫做主表（父表），属于主从关系。

其中主表中的关联字段叫做主键，外表中的关联字段叫做外键。

外键约束主要作用就是能够让表与表之间的数据建立关联，使数据更加完整，关联性更强。关于完整性，可以通过一个例子来说明假如有两种表，一张用户账户表（用于存储用户账户），一张是账户信息表（用于存储账户中的信息）。

1）我不小心将用户账户表中的某个用户删除了，那么账户信息表中与这个用户有关的数据就变成无源数据了，找不到其属于哪个用户账户，导致用户信息不完整。

2）我在账户信息表中随便添加了一条数据，而其在用户账户表中没有对应的用户，这样用户信息也是不完整的。

如果有了外键约束，将用户账户表设为主表，账户信息表设为从表，那么就无法直接往账户信息表中插入数据，在用户账户表中删除用户，账户信息表中的用户信息也会被删除。外键约束的方式可以使两张紧密的联系在一起，保障数据完整性和一致性的同时，日常维护也更加方便。

语法：

- 添加外键：

  ~~~sql
  CREATE TABLE 表名(
  	字段名 数据类型,
      ...
      [CONSTRAINT] 外键名称 FOREIGN KEY (外键字段名) REFERENCES 主表(主表列名)
  );
  
  ALTER TABLE 表名 ADD CONSTRAINT 外键名称 FOREIGN KEY (外键字段名) REFERENCES 主表(主表列名);
  ~~~

  

- 删除外键

  ~~~sql
  ALTER TABLE 表名 DROP FOREIGN KEY 外键名称;
  ~~~

  

外键约束：删除/更新行为

| 行为      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| NO ACTION | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。(与RESTRICT一致) |
| RESTRICT  | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。(与NO ACTION一致) |
| CASCADE   | 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有，则也删除/更新外键在子表中的记录。 |
| SET NULL  | 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null（这就要求该外键字段允许取null）。 |

添加外键约束：

~~~ sql
ALTER TABLE 表名 ADD CONSTRAINT 外键名称 FOREIGN KEY (外键字段名) REFERENCES 主表(主表列名) ON UPDATE CASCADE ON DELETE CASCADE;
~~~



## 多表查询

### 多表关系

项目开发中，在进行数据库表结构设计时，会根据业务需求及业务模块之间的关系，分析并设计表结构，由于业务之间相互关联，所以各个表结构之间也存在着各种联系，基本上分为三种：

1. 一对多（多对一）：部门表-员工表。
2. 多对多：学生-课程，一个学生可以选择多门课程。
3. 一对一：用户-用户详情信息，多用于单表拆分，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中。



### 多表查询概述

概述：指从多张表中查询数据。

笛卡尔积：笛卡尔乘积是指在数学中，两个集合A集合和B集合的所有组合情况。（在多表查询时，需要消除无效的笛卡尔积)

多表查询分类：

- 连接查询：
  - 内连接：相当于查询A、B交集部分数据
  - 外连接：
    - 左外连接：查询左表所有数据，以及两张表交集部分数据
    - 右外连接：查询右表所有数据，以及两张表交集部分数据
  - 自连接：当前表与自身的连接查询，自连接必须使用表别名
- 子查询

### 内连接

语法：

- 隐式内连接

  ~~~sql
  SELECT 字段列表 FROM 表1,表2 WHERE 条件 ...;
  ~~~

  

- 显式内连接

  ~~~sql
  SELECT 字段列表 FROM 表1 [INNER] JOIN 表2 ON 连接条件 ...;
  ~~~

  

### 外连接

语法：

- 左外连接：查询表1的所有数据以及和表2交集部分的数据。

  ~~~sql
  SELECT 字段列表 FROM 表1 LEFT [outer] JOIN 表2 ON 连接条件 ...;
  ~~~

  

- 有外连接：查询表2的所有数据以及和表1交集部分的数据。

  ~~~sql
  SELECT 字段列表 FROM 表1 RIGHT [outer] JOIN 表2 ON 连接条件 ...;
  ~~~

  

### 自连接

语法：

~~~sql
SELECT 字段列表 FROM 表1 别名A JOIN 表1 别名B ON 条件 ...;
~~~



> 注意：自连接查询，可以是内连接查询，也可以是外连接查询。

示例：

一个表中有员工信息（id），又有员工的领导信息(managerid)，查询员工，和员工的领导。

~~~sql
select a.name,b.name from user a left join user b on a.id = b.managerid;
~~~



### 链接查询

对于union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集。

~~~sql
SELECT 字段列表 FROM 表A
union [all]
SELECT 字段列表	FROM 表B;
~~~



> 注意：
>
> - 对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。
> - union all会将全部的数据直接合并在一起，union会对合并之后的数据去重。

### 子查询

- SQL语句中嵌套SELECT语句，称为嵌套查询，又称子查询。

  ~~~sql
  SELECT 字段列表 FROM t1 WHERE column = (SELECT column1 FROM t2);
  ~~~

  

  > 注意：子查询外部查询的语句是 INSERT/UPDATE/SELECT的任何一个。

- 根据子查询结果不同，分为：

  - 标量子查询（子查询结果为单个值）
  - 列子查询(子查询结果为一列)
  - 行子查询（子查询结果为一行）
  - 表子查询（子查询结果为多行多列）

- 根据子查询位置，分为：WHERE之后、FROM之后、SELECT之后。

#### 标量子查询

子查询返回的结果是单个值（数字、字符串、日期等），最简单的形式，这种子查询成为标量子查询。

常用的操作符：= <> < =< > =>

示例：

查询小王入职后的人员名单

~~~sql
# a.查询小王入职的日期
select entrydate from emp where name = "小王";
# b.查询制定入职日期后的人员名单
select * from emp where entrydate > 2022-02-11;

select * from emp where entrydate > (select entrydate from emp where name = "小王");
~~~



#### 列子查询

子查询返回的结果是一列（可以是多行），这种子查询称为列子查询。

常用的操作符：IN、NOTIN、ANY、SOME、ALL

| 操作符 | 描述                                   |
| ------ | -------------------------------------- |
| IN     | 在指定的集合范围之内，多选一           |
| NOT IN | 不在指定的集合范围内                   |
| ANY    | 子查询返回列表，有任意一个满足即可     |
| SOME   | 与ANY等同，使用SOME的地方都可以使用ANY |
| ALL    | 子查询返回列表的所有值都必须要满足     |

示例：

查看研发部和总经办的所有员工信息

~~~sql
select * from user u left join dept de on u.dept_id = de.id where dept_id in (select id from dept where name ='研发部' or name='总经办') ;
~~~



查询比研发部其中一个人工资高的员工信息

~~~sql
select * from user where salary >= some (select salary from user where dept_id = (select id from dept where name='研发部'));
~~~



#### 行子查询

子查询返回的结果是一行（可以是多列），这种子查询称为行子查询。	

常用的操作符：=、<>、IN、NOT IN



#### 表子查询

示例：

查询入职时间是"2006-07-01"之后的员工信息，以及部门信息：

~~~sql
select e.*,d.* from (select * from emp where emp.entrydate > "2006-07-01") e left join dept d on e.dept_id = d.id;
~~~



## 事务

### 四大特性

ACID

- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。
- 一致性（consistency）：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

### 并发事务问题

| 问题       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 脏读       | 一个事务读到另外一个事务还没有提交的数据。                   |
| 不可重复读 | 一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读。 |
| 幻读       | 一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了”幻影”。 |

### 事务隔离级别

| 隔离级别              | 脏读  | 不可重复读 | 幻读  |
| --------------------- | ----- | ---------- | ----- |
| Read uncommitted      | true  | true       | true  |
| Read committed        | false | true       | true  |
| Repeatable Read(默认) | false | false      | true  |
| Serializable          | false | false      | false |

true代表会当前隔离级别会出现此问题。



隔离级别操作

~~~mysql
# 查看事务隔离级别
select @@TRANSACTION_ISOLATION;
# 设置隔离级别
SET [SESSIONI|GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
~~~



事务操作

~~~mysql
# 开启事务
START TRANSACTION; 

# 提交/回滚事务
COMMIT / ROLLBACK;
~~~



## 索引

### 概述

索引（index）是帮助MySQL`高效获取数据`的`数据结构`（有序）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引。

### 结构

MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的结构，主要包含以下几种：

| 索引结构              | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| **B+Tree 结构**       | **最常见的索引类型，大部分引擎都支持B+树索引**               |
| Hash 索引             | 底层数据结构是用哈希表实现的，只有精确匹配索引列的查询才有效，不支持范围查询 |
| R-Tree（空间索引）    | 空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-text（全文索引） | 是一种通过建立倒排索引，快速匹配文档的方式。类似于Lucene,Solr,ES |

> 我们平常所说的索引，如果没有特别指明都是指B+树结构组织的索引。

为什么InnoDB存储引l擎选择使用B+tree索引结构？

- 相对于二叉树，层级更少，搜索效率高；
- 对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低;
- 相对Hash索引，B+tree支持范围匹配及排序操作；

### 分类

![image-20241130204358344](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20241130204358344.png)

在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种：

![image-20241130204526798](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20241130204526798.png)

聚集索引选取规则：

- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一（UNIQUE）索I为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。

### 语法

创建索引

~~~mysql
CREATE [ UNIQUE | FULLTEXT ] INDEX index_name ON table_name (index_col_name,...);
~~~



查看索引

~~~mysql
SHOW INDEX FROM table_name;
~~~



删除索引

~~~mysql
DROP INDEX index_name ON table_name;
~~~





### SQL性能分析

- SQL执行频率

  MySQL 客户端连接成功后，通过 show  [session|global] status 命令可以提供服务器状态信息。通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次：

  ~~~mysql
  show global status like 'Com_______';
  ~~~

  

- 慢查询日志

  慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SQL语句的日志。 

  MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息：

  ~~~shell
  # 开启MySQL慢日志查询开关 
  slow_query_log=1
  
  # 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志 
  long_query_time=2
  ~~~

  配置完毕之后，通过以下指令重新启动MysQL服务器进行测试，查看慢日志文件中记录的信息/var/lib/mysql/localhost-slow.log。

- profile

  show profiles能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。通过have_profiling参数，能够看到当前MySQL是否支持。

  profile操作:

  ~~~mysql
  SELECT @@HAVE_profiling;
  ~~~

  默认profiling是关闭的，可以通过set语句在session/global级别开启profiling：

  ~~~mysql
  SET profiling = 1;
  ~~~

  执行一系列的业务SQL的操作，然后通过如下指令查看指令的执行耗时：

  ~~~mysql
  # 查看每一条SQL的耗时基本情况 
  show profiles;
  
  # 查看指定query_id的sQL语句各个阶段的耗时情况 
  show profile for query query_id;
  
  # 查看指定queryid的SQL语句CPU的使用情况 
  show profile cpu for query query_id;
  ~~~

- explain 执行计划

  EXPLAIN 执行计划各字段含义：

  - id

    select查询的序列号，表示查询中执行select子句或者是操作表的顺序(id相同，执行顺序上到下；id不同，值越大，越先执行)。

  - select_type

    表示SELECT的类型，常见的取值有SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、 UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（SELECT/WHERE之后包含了子查询）等

  - type

    表示连接类型，性能由好到差的连接类型为NuLL、system、const、eq_ref、ref、range、index、all。

  - possible_key

    显示可能应用在这张表上的索引，一个或多个。

  - Key

    实际使用的索引，如果为NULL，则没有使用索引。

  - Key_len

    表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好。

  - rows

    MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值，可能并不总是准确的。

  - filtered

    表示返回结果的行数占需读取行数的百分比，filtered的值越大越好。

​	

### 索引使用

- 最左前缀法则

  如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，索引将部分失效(后面的字段索引失效)。

- 范围查询

  联合索引中，出现范围查询（>，<)，范围查询右侧的列索引失效。

- 索引列运算

  不要在索引列上进行运算操作（函数等），索引将失效。

  ~~~shell
  select * from tb user where substring(phone,10,2) ='15';
  ~~~

  

- 字符串不加引号

  字符串类型字段使用时，不加引号，索引将失效。

- 模糊查询

  如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

  ~~~shell
  explain select * from tb_user where profession like '&I&';
  ~~~

- or连接的条件
  用or分割开的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

- 数据分布影响

  如果MySQL评估使用索引I比全表更慢，则不使用索引。

- 单列索引于联合索引

  单列索引：即一个索引包含单个列。

  联合索引：即一个索引包含了多个列。

  在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。

- SQL 提示

  SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

  ~~~mysql
  # use index
  explain select * from tb_user use index(idx_user_pro) where profession = '软件工程'; 
  # ignore index
  explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程'; 
  # force index
  explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';
  ~~~

  

- 覆盖索引

  尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少select *。

- 前缀索引

  当字段类型为字符串（varchar，text等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘lO，影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

  语法

  ~~~mysql
  create index idx_xxxx on table_name(column(n));
  ~~~

  可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高，唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

  ~~~mysql
  select count(distinctemail) / count(*) from tb_user ;
  select count(distinct substring(email,1,5) / count(*) from tb_user ;
  ~~~

  

  

### 索引设计原则

1. 针对于数据量较大，且查询比较频繁的表建立索引。
2. 针对于常作为查询条件（where）、排序（orderby）、分组（groupby）操作的字段建立索引。
3. 量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。例如：性别，区分度不高
4. 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7. 如果索引列不能存储NULL值，请在创建表时使用NOTNULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。



## SQL 优化

### 插入数据

- insert 优化

  1. 批量插入

     ~~~shell
     insert into tb_table values(1,'tom'),(2,'cat'),(3,'jerry');
     ~~~

     

  2. 手动提交事务

     ~~~shell
     start transaction;
     insert into tb_table values(1,'tom'),(2,'cat'),(3,'jerry');
     commit;
     ~~~

     

  3. 主键顺序插入

     ~~~shell
     # 主键乱序插入：
     3 7 5 89 15 4 2 88 21 9 1 8 
     # 主键顺序插入：
     1 2 3 4 5 6 7 8 9 15 21 88 89
     ~~~

     

  4. 如果一次性需要插入大批量数据，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令进行插入。

     ~~~shell
     # 客户端连接服务端时，加上参数--local-infile 
     mysql --local-infile -uroot -p
     # 设置全局参数locaL_infile为l，开启从本地加载文件导入数据的开关 
     set global local_infile=1;
     # 执行load指令将准备好的数据，加载到表结构中
     load data local infile '/root/sqll.log' into table tb_user fields terminated by ',' lines terminated by '\n';
     ~~~


### 主键优化

- 数据组织方式

  在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表(index organized table IOT)。

  ![image-20241228105942959](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20241228105942959.png)

  

- 页分裂

  页可以为空，也可以填充一半，也可以填充100%。每个页包含了2-N行数据（如果一行数据多大，会行溢出)，根据主键排列。

- 页合并

  当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。

  当页中删除的记录达到 MERGE_THRESHOLD（默认为页的50%），InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优。

  > 知识小贴士：
  > 	MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引I时指定。

- 主键设计原则

  - 满足业务需求的情况下，尽量降低主键的长度。

  - 插入数据时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键。

  - 尽量不要使用UUID做主键或者是其他自然主键，如身份证号。
  - 业务操作时，避免对主键的修改。

### order by优化

1. Using filesort:通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort排序。
2. Using index：通过有序索引l顺序扫描直接返回有序数据，这种情况即为usingindex，不需要额外排序，操作效率高。

### group by 优化

- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
- 尽量使用覆盖索引。
- 多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。
- 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小sort_buffer_size(默认256k)。

### limit 优化

一个常见又非常头疼的问题就是limit2000000,10，此时需要MySQL排序前2000010记录，仅仅返回2000000－2000010 的记录，其他记录丢弃，查询排序的代价非常大。

优化思路：一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。

~~~sql
explain select * from tb_sku t , (select id from tb_sku order by id limit 2000000,10) a where t.id = a.id;
~~~



### count 优化

MyISAM引擎把一个表的总行数存在了磁盘上，因此执行count（*）的时候会直接返回这个数，效率很高；

InnoDB引擎就麻烦了，它执行count（*）的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。

按照效率排序的话，count(字段)count(主键id)<count(1)~count(*)，所以尽量使用count(*)。

### update 优化

InnoDB的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则会从行锁升级为表锁。



## 视图/存储过程/触发器

### 视图

视图一方面可以帮我们使用表的一部分而不是所有的表，另一方面也可以针对不同的用户制定不同的查询视图比如，针对一个公司的销售人员，我们只想给他看部分数据，而某些特殊的数据，比如采购的价格，则不会提供给他。再比如，人员薪酬是个敏感的字段，那么只给某个级别以上的人员开放，其他人的查询视图中则不提供这个字段。

理解：

- 视图是一种虚拟表，本身是不具有数据的，占用很少的内存空间，它是SQL中的一个重要概念。
- 视图建立在已有表的基础上，视图赖以建立的这些表称为基表。

#### 语句

创建

~~~sql
CREATE [OR REPLACE]
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW 视图名称 [(字段列表)]
AS 查询语句
[WITH [CASCADED|LOCAL] CHECK OPTION]
~~~



查询

~~~mysql
# 查看创建视图语句
SHOW CREATE VIEW 视图名称;

# 查看视图数据
SELECT * FROM 视图名称...;

# 修改
## 方法一：
CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 WITH[CASCADED|LOCAL]CHECK OPTION];
## 方法二：
ALTER VIEW 视图名称[(列名列表)] AS SELECT语句 [WITH[CASCADED|LOCAL] CHECK OPTION];

# 删除
DROP VIEW [IF EXISTS] 视图名称 [视图名称]...;
~~~



#### 创建视图

1. 创建单表视图

   ~~~mysql
   create VIEW emp1(emp1_id,emp1_name)
   AS
   SELECT id,name from sql_test;
   ~~~

   

2. 创建多表联合视图

   ~~~mysql
   CREATE VIEW tb_stu_course_view AS 
   select * from student s,students_course sc,course c where s.id = sc.studentid and sc.studentid = c.id;
   ~~~



### 存储过程

存储过程是事先经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。

存储过程思想上很简单，就是数据库SQL语言层面的代码封装与重用。



### 触发器

触发器是与表有关的数据库对象，指在insert/update/delete之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。

使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句
级触发。

#### 语法

创建

~~~sql
CREATE TRIGGER trigger_name
BEFORE/AFTER INSERT/UPDATE/DELETE
ON tbl_name FOR EACH ROW     # 行级触发器
BEGIN
	trigger_stmt;
END;
~~~

查看

~~~sql
SHOW TRIGGER;
~~~

删除

~~~sql
DROP TRIGGER [schema_name.]trigger_name;	# 如果没有指定 schema_name，默认为当前数据库。
~~~



## 锁

锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。



### 全局锁

全局锁就是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的DML的写语句，DDL语句，已经更新操作的事务提交语句都将被阻塞。
其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。

特点：

1. 如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆。
2. 如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志（binlog），会导致主从延迟。

> 小卫士：
>
> 在lnnoDB引擎中，我们可以在备份时加上参数 --single-transaction 参数来完成不加锁的一致性数据备份。

语法：

~~~sql
#　加全局锁
flush tables with read lock;
# 解（释放）锁
unlock tables;
~~~



### 表级锁

表级锁，每次作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中。

对于表级锁，分为三类：

1. 表锁

   1. 表共享读锁（read lock）
      1. 读锁不会阻塞其他客户端的读，但是会阻塞写。
   2. 表独占写锁（write lock）
      1. 写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写。

2. 元数据锁（meta data lock，MDL）

   MDL加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。MDL锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作。为了避免DML与DDL冲突，保证读写的正确性。

   在MySQL5.5中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁(共享)；当对表结构进行变更操作的时候，加MDL写锁(排他)。

   | 对应SQL                                       | 锁类型                                | 说明                                             |
   | --------------------------------------------- | ------------------------------------- | ------------------------------------------------ |
   | lock tables xxx read / write                  | SHARED_READ_ONLY/SHARED_NO_READ_WRITE |                                                  |
   | select，select ... lock in share mode         | SHARED_READ                           | 与SHARED_READ、SHARED_WRITE兼容，与EXCLUSIVE互斥 |
   | insert，update，delete，select ... for update | SHARED_WRITE                          | 与SHARED_READ、SHARED_WRITE兼容，与EXCLUSIVE互斥 |
   | alter table ...                               | EXCLUSIVE                             | 与其他的MDL都互斥                                |

   查看元数据锁：

   ~~~sql
   select object_type,object_schema,object_name,lock_type,lock_duration from performance_schema.metadata_locks;
   ~~~

   

3. 意向锁

   为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

   1. 意向共享锁（Is）：由语句select ... lock in share mode添加。	

      与表锁共享锁（read）兼容，与表锁排它锁（write）互斥。

   2. 意向排他锁（Ix）：由insert、update、delete、select ... for update添加。

      与表锁共享锁（read）及排它锁（write）都互斥。意向锁之间不会互斥。

   

语法：

~~~sql
# 加锁	
lock tables [表名...] read/write;

# 释放锁
unlock tables / 客户端连接关闭
~~~



### 行级锁

行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在InnoDB存储引擎中。

InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加的锁。对于行级锁，主要分为以下三类：

1. 行锁（RecordLock）：锁定单个行记录的锁，防止其他事务对此行进行update和delete。在RC、RR隔离级别下都支持。
2. 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。在RR隔离级别下都支持。
3. 临建锁（Next-Key Lock）：锁定索引记录本身以及索引记录之间的间隙，从而防止其他事务在锁定范围内插入新记录，避免幻读（Phantom Read）问题。



默认情况下，InnoDB在REPEATABLEREAD事务隔离级别运行，InnoDB使用next-key锁进行搜索和索引扫描，以防止幻读。

行锁-演示

1. 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。
2. InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，此时就会升级为表锁。



间隙锁/临键锁-演示

1. 索引上的等值查询（唯一索引)，给不存在的记录加锁时，优化为间隙锁。
2. 索引上的等值查询（普通索引)，向右遍历时最后一个值不满足查询需求时，next-keylock退化为间隙锁。
3. 索引上的范围查询（唯一索引)--会访问到不满足条件的第一个值为止。

> 注意：间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。

## InnoDB引擎

inoDB存储引擎，是mysql5.5版本后（不包括5.5）的默认的存储引擎。

![image-20241130113200740](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20241130113200740.png)

### mysql 逻辑存储架构

![image-20250113104706496](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20250113104706496.png)

- 表空间（TableSpace）

  表空间（ibd文件），一个mysql实例可以对应多个表空间，用于存储记录、索引等数据。

  默认文件存放位置 `/var/lib/mysql`。

- 段（Segment）

  段，分为数据段（Leafnode segment）、索引段（Non-leafnode segment）、回滚段（Rollback segment），InnoDB 是索引组织表，数据段就是B+树的叶子节点，索引段即为B+树的非叶子节点。段用来管理多个Extent（区）。

- 区（Extent）

  区，表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区中一共有64个连续的页。

- 页（page）

  页，是InnoDB存储引擎磁盘管理的最小单元，每个页的大小默认为16KB。为了保证页的连续性，InnoDB存储引擎每次从磁盘申请4-5个区。

- 行（row）

  行，InnoDB存储引擎数据是按行进行存放的。	

  - Trx_id：每次对某条记录进行改动时，都会把对应的事务id赋值给trx_id隐藏列。
  - Roll_pointer 每次对某条引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型。

查看引擎

~~~mysql
SHOW ENGINES;
~~~



### 架构

MySQL5.5版本开始，默认使用InnoDB存储引擎，侧为磁盘结构。
具有崩溃恢复特性，在日常开发中使用非常广泛。下面是InnoDB架构图，左侧为内存结构，右它擅长事务处理。

![image-20250113110039091](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20250113110039091.png)

#### 内存

- BufferPool：缓冲池是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘I0，加快处理速度。

- 缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型：

  - free page：空闲page，未被使用。
  - cleanpage：被使用page，数据没有被修改过。
  - dirtypage：脏页，被使用page，数据被修改过，也中数据与磁盘的数据产生了不一致。

- Change Buffer；更改缓冲区(针对于非唯一二级索引页) ,在执行DML语句时,如果这些数据Page 没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区 Change Buffer中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中。

- AdaptiveHashIndex：自适应hash索引，用于优化对BufferPool数据的查询。InnoDB存储引擎会监控对表上各索引页的查询，如果观察到hash索引可以提升速度，则建立hash索引，称之为自适应hash 索引。
  **自适应哈希索引，无需人工干预，是系统根据情况自动完成。**

  参数：adaptive_hash_index

- LogBuffer：日志缓冲区，用来保存要写入到磁盘中的log日志数据（redolog、undolg），默认大小为16MB，日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘1/0。

  参数：

  - innodb_log_buffer_size：缓冲区大小
  - innodb_flush_log_at_trx_commit：日志刷新到磁盘时机
    - 1：日志在每次事务提交时写入并刷新到磁盘 
    - 0：每秒将日志写入并刷新到磁盘一次。
    - 2：日志在每次事务提交后写入，并每秒刷新到磁盘一次。

#### 磁盘

- System Tablespace：系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据。（在MySQL5.x版本中还包含lnnoDB数据字典、undolog等)

  参数：innodb_data_file_path

- File-Per-Table Tablespaces：每个表的文件表空间包含单个lnnoDB表的数据和索引，并存储在文件系统上的单个数据文件中。

  参数：innodb_file_per_table

- General Tablespaces：通用表空间，需要通过CREATETABLESPACE语法创建通用表空间，在创建表时，可以指定该表空间。

- Undo Tablespaces：撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小 16M），用于存储undolog日志。

- Temporary Tablespaces：InnoDB使用会话临时表空间和全局临时表空间。存储用户创建的临时表等数据。

- Doublewrite Buffer Files：双写缓冲区，innoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件中，便于系统异常时恢复数据。

  文件

  - #ib_16384_0.dblwr
  - #ib_16384_1.dblwr

- Redo Log：重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redologbuffer）以及重做日志文件（redolog），前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中，用于在刷新脏页到磁盘时，发生错误时，进行数据恢复使用。

  以循环方式写入重做日志文件，涉及两个文件：

  - ib_logfile0
  - ib_logfile1

#### 后台线程

![image-20250113163349418](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20250113163349418.png)

1. Master Thread

   核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中，保持数据的一致性，还包括脏页的刷新、合并插入缓存、undo页的回收。

2. IO Thread

   在InnoDB存储引擎中大量使用了AIO来处理lO请求，这样可以极大地提高数据库的性能，而lO Thread主要负责这些lO请求的回调。

   | 线程类型             | 默认个数 | 职责                         |
   | -------------------- | -------- | ---------------------------- |
   | Read thread          | 4        | 负责读操作                   |
   | Write thread         | 4        | 负责写操作                   |
   | Log thread           | 1        | 负责将日志缓冲区刷新到磁盘   |
   | Insert buffer thread | 1        | 负责将写缓冲区内容刷新到磁盘 |

3. Purge Thread

   主要用于回收事务已经提交了的undolog，在事务提交之后，undolog 可能不用了，就用它来回收。

4. Page Cleaner Thread

   协助Master Thread 刷新脏页到磁盘的线程，它可以减轻Master Thread 的工作压力，减少阻塞。

### 事务原理

事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

![image-20250114105641016](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20250114105641016.png)

持久性：Redo Log：重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redologbuffer）以及重做日志文件（redolog），前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中，用于在刷新脏页到磁盘时，发生错误时，进行数据恢复使用。

原子性：undo log：回滚日志，用于记录数据被修改前的信息，作用包含两个：提供回滚和MVCC(多版本并发控制）。
undo log和redo log记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undolog中的逻辑记录读取到相应的内容并进行回滚。
Undolog销毁：undo log在事务执行时产生，事务提交时，并不会立即删除undolog，因为这些日志可能还用于Mvcc。
Undo log存储：undo log采用段的方式进行管理和记录，存放在前面介绍的rollback segment回滚段中，内部包含1024个undo log segment.

### MVCC

- 当前读

  读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如： select ...lockin share mode(共享锁)，select ...forupdate、update、insert、delete(排他锁)都是一种当前读。

- 快照读

  简单的select（不加锁）就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。

  - Read committed：每次seletct，都会生成一个快照。
  - Repeatable Read：开启事务后第一个select语句才是快照读的地方。
  - Serializable：快照读会退化成当前读。

- MVCC

  全称 Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MysQL实现MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undolog日志、readView。

#### 实现原理

- 隐藏字段

  1. DB_TRX_ID：最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID。
  2. DB_ROLL_PTR：回滚指针，指向这条记录的上一个版本，用于配合undolog，指向上一个版本。
  3. DB_ROW_ID：隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段。

  > 查看表空间文件详细信息（包含隐藏字段）：`ibd2sdi <表空间文件>`

- undo log

  回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。
  当insert的时候，产生的undolog日志只在回滚时需要，在事务提交后，可被立即删除。
  而update、delete的时候，产生的undolog日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。

  undo log 版本链

  ![image-20250114153224377](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20250114153224377.png)

  - 不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录。

- readView

  ReadView（读视图）是快照读SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id。

  - ReadView中包含了四个核心字段：

    | 字段           | 含义                                                 |
    | -------------- | ---------------------------------------------------- |
    | m_ids          | 当前活跃的事务ID集合                                 |
    | min_trx_id     | 最小活跃事务ID                                       |
    | max_trx_id     | 预分配事务ID，当前最大事务ID+1（因为事务ID是自增的） |
    | creator_trx_id | ReadView创建者的事务ID                               |

  - 版本链数据访问规则：

    trx_id：代表是当前事务ID。

    - trx_id ==creator_trx_id？可以访问该版本

      成立，说明数据是当前这个事务更改的。

    - trx_id < min_trx_id？可以访问该版本

      成立，说明数据已经提交了。

    - trx_id > max_trx_id? 不可以访问该版本

      成立，说明该事务是在ReadView生成后才开启。

    - min_trx_id<=trx_id<=max_trx_id？如果trx_id不在m_ids中是可以访问该版本的
      成立，说明数据已经提交。

  > 不同的隔离级别，生成ReadView的时机不同：
  >
  > - READ COMMITTED：在事务中每一次执行快照读时生成ReadVieW。
  > - REPEATABLE READ：仅在事务中第一次执行快照读时生成ReadVieW，后续复用该ReadVieW。

## MySQL管理

### 系统数据库

Mysql数据库安装完成后，自带了一下四个数据库，具体作用如下：

| 数据库             | 含义                                                         |
| ------------------ | ------------------------------------------------------------ |
| mysql              | 存储MySQL服务器正常运行所需要的各种信息（时区、主从、用户、权限等） |
| information_schema | 提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等 |
| performance_schema | 为MySQL服务器运行时状态提供了一个底层监控功能，主要用于收集数据库服务器性能参数 |
| sys                | 包含了一系列方便DBA和开发人员利用performance_schema性能数据库进行性能调优和诊断的视图 |

### 常用工具

- mysql

  该mysql不是指mysql服务，而是指mysql的客户端工具。

  ~~~sql
  # 语法
  	mysql [options] [database]
  # 选项
  	-u,--user=name 			# 执行用户名
  	-p,--password[=name]	# 指定密码
  	-h,--host=name			# 指定服务器
  	-P,--Port=port			# 指定连接端口
  	-e,--execute=name		# 执行SQL语句并退出
  ~~~

  

- mysqladmin

  mysqladmin是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。

  示例：

  ~~~shell
  # 查看数据库版本
  mysqladmin -uroot -p123456 version
  
  # 查看系统变量
  mysqladmin -uroot -p123456 variables	
  ~~~

- mysqlbinlog

  由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog日志管理工具。

  ~~~shell
  # 语法
  	mysqlbinlog [options] log-file1 log-file2 ...
  # 选项
  	-d,--database=name 		# 指定数据库名称，只能列出指定数据库相关操作
  	-o,--offset=#			# 忽略掉日志中的前n行命令
  	-r,--result-file=name	# 将输出的文本格式日志输出到指定文件
  	-s,--short-from			# 显示简单格式，省略掉一些信息
  	-start-datatime=date1 --stop-datatime=date2		# 指定日期间隔内的所有日志
  	-start-position=pos1 --stop-position=pos2		# 指定位置间隔内的所有日志
  ~~~

- mysqlshow

  mysqlshow客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引。

  ~~~shell
  # 语法
  	mysqlshow [options] [db_name [table_name[col_name]]]
  # 选项
  	--count		# 显示数据库及表的统计信息（数据库，表均可以不指定）
  	-i			# 显示指定数据库或者指定表的状态信息
  ~~~

- mysqldump

  mysqldump客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的SQL语句。

  ~~~shell
  # 语法：
  	mysqldump [options] db_name [tables]
  	mysqldump [options]	--database/-B [db2 db3...]
  	mysqldump [options] --all-databases/-A
  # 连接选项:
  	-u,--user=name			# 指定用户名
  	-p,--password[=name]	# 指定密码
  	-h,--host=name			# 指定服务器
  	-P,--port=#				# 指定连接端口
  # 输出选项:
  	--add-drop-database			# 在每一个数据库创建语句前加上 drop database 语句
  	--add-drop-table			# 在每一表创建语句前加上droptable语句，默认开启；不开启（--skip-add-drop-table）
  	-n,--no-create-db			# 不包含数据库的创建语句
  	-t,--no-create-info			# 不包含数据表的创建语句
  	-d,--no-data				# 不包含数据
  	-T,--tab=name				# 自动生成两个文件：一个.sgl文件，创建表结构的语句；一个.txt文件，数据文件
  ~~~

- mysqlimport/source

  mysqlimport 是客户端数据导入工具，用来导入mysqldump 加-T参数后导出的文本文件。

  ~~~sql
  # 语法：
  	mysqlimport [options] db_name textfile1 [textfile2]
  # 示例：
  	mysqlimport -uroot test1 /tmp/city.txt
  ~~~

  

  如果需要导入sql文件，可以使用mysql中的source指令：

  ~~~sql
  # 语法：
  	source /root/xxxx.sql
  ~~~

## 日志

### 错误日志

错误日志是MySQL中最重要的日志之一，它记录了当mysqld启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。当数据库出现任何故障导致无法正常使用时，建议首先查看此日志。

该日志是默认开启的，默认存放目录/var/log/，默认的日志文件名为mysgld.log。查看日志位置：

~~~sql
show variables like "%log_error%"
~~~



### 二进制日志

### 查询日志

### 慢查询日志



## 主从复制





## 读写分离





## 分库分表

