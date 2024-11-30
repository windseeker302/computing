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
  SELECT * FROM user;
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

## 图形化界面

- sqlyog
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



## 视图/存储过程/触发器

### 视图

视图一方面可以帮我们使用表的一部分而不是所有的表，另一方面也可以针对不同的用户制定不同的查询视图比如，针对一个公司的销售人员，我们只想给他看部分数据，而某些特殊的数据，比如采购的价格，则不会提供给他。再比如，人员薪酬是个敏感的字段，那么只给某个级别以上的人员开放，其他人的查询视图中则不提供这个字段。

理解：

- 视图是一种虚拟表，本身是不具有数据的，占用很少的内存空间，它是SQL中的一个重要概念。
- 视图建立在已有表的基础上，视图赖以建立的这些表称为基表。

#### 语句

~~~mysql
CREATE [OR REPLACE]
[ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
VIEW 视图名称 [(字段列表)]
AS 查询语句
[WITH [CASCADED|LOCAL] CHECK OPTION]
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
   ~~~

   

3. 基于视图创建视图

   

   

   



### 存储过程

### 触发器



## 锁

## InnoDB引擎

## 日志

## 主从复制

## 读写分离

## 分库分表

