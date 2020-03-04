---
title: Java学习笔记（十四）：MySQL数据库
date: 2020-1-13 13:36
urlname: 2020011301
categories: Java
tags:
  - Java
  - MySQL
author: foochane
toc: true
mathjax: true
top: true
top_img: /images/background/16.jpg
cover: 
---


<!-- 

>[foochane](https://foochane.cn/) ：[https://foochane.cn/article/2020011301.html](https://foochane.cn/article/2020011301.html) 

-->



## 1 数据库的基本概念

### 1.1 什么是数据库

数据库（DataBase,简称：DB），是一个**用于存储和管理数据的仓库**。



### 1.2 数据库的特定

- 持久化存储数据。数据库其实是一个**文件系统**。
- 方便存储和管理数据。
- 使用统一的方式操作数据库——SQL。



### 1.3 常用的数据库

可以通过网址：[https://db-engines.com/en/ranking](https://db-engines.com/en/ranking)   ，    查看数据库的排行榜。

![](https://static.cnbetacdn.com/article/2019/1104/a4263524307eb93.png)



## 2 SQL

### 2.1 SQL的概念

SQL（Structured Query Language）: 结构化查询语义，就是定义了操作所有关系型数据库的规则。



### 2.2 SQL的通用语法

1. SQL 语句可以单行或者多行书写，以分号 `“;”`结尾。

2. 可以使用空格和缩进来增强语句的可以读性。

3. MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。

4. SQL语句的注释：

   - 单行注释：

     ```sql
     -- 注释内容
     
     # 注释内容（MySQL特有）
     ```

   - 多行注释

     ```mysql
     /* 
     注释
     注释
     */
     ```

     

### 2.3 SQL的分类

1. DDL（Data Definition Language），数据定义语义：

   用来定义数据库对象：数据库、表、列等。关键字：`create`、`drop`、`alert`等。

2. DML（Data Manipulation Language），数据操作语义：
   用来对数据库中的表进行**增删改**。关键字：`insert`、`delete`、`update`等

3. DQL（Data Query Language），数据查询语言：

   用来查询数据库中表的数据。关键字：`select`、`where`等。

4. DCL（Data Control Language），数据控制语言：

   用来定义数据库的访问权限和安全级别，以及创建用户。关键字：`GRANT`,`REVOKE`等。

### 2.4 CRUD

- C（Create）：创建
- R（Retrieve）：查询
- U（Update）：修改
- D（Delete）：删除



## 4 操作数据库

### 4.1 创建数据库



```sql
# 创建数据库
create database db1;

# 创建数据库，判断不存在，再创建
create database if not exists db2;

# 创建数据库，并指定字符集
create database db3 character set gbk;

# 创建数据库，判断是否存在，并指定字符集
create database if not exists db4 character set gbk;
```



### 4.2 查询数据库

#### 查询所有的数据库名称

```sql
# 查询所有的数据库名称
show databases;
```

示例：

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| education_course   |
| education_system   |
| education_user     |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
7 rows in set (0.01 sec)
```

#### 查询某个数据库的字符集

#### 查询某个数据库的创建语句

```sql
# 查询某个数据库的字符集/查询某个数据库的创建语句
show create database 数据库名称
```



示例：

```sql
mysql> show create database mysql;
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| mysql    | CREATE DATABASE `mysql` /*!40100 DEFAULT CHARACTER SET latin1 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)
```



### 4.3 修改数据库

#### 修改数据库的字符集

```sql
# 修改字符集
alter database 数据库名称 character set 字符集名称;
```

示例：

```sql
mysql> alter database db3 character set utf8;
Query OK, 1 row affected (0.00 sec)

mysql> show create database db3;
+----------+--------------------------------------------------------------+
| Database | Create Database                                              |
+----------+--------------------------------------------------------------+
| db3      | CREATE DATABASE `db3` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+----------+--------------------------------------------------------------+
1 row in set (0.00 sec)
```



### 4.4 删除数据库



```sql
#删除数据库
drop dababase 数据库名称;

# 先判断是否存在，再删除
drop database if exists 数据库名称;
```



### 4.5 使用数据库

#### 查询当前正在使用的数据库

```sql
# 查看当前使用的数据库
select database();
```



#### 使用数据库

```sql
# 使用数据库
use 数据库名称;
```



## 5 操作表


### 5.1 创建表

#### 创建语法

```sql
create tabele 表名(
	列名1 数据类型1,
    列名2 数据类型2,
    ......
    列名n 数据类型n
)
```



示例：

```sql
create table student(
	id int,
    name varchar(32),
    age int,
    score double(4,1), #最多4位，取1位小数（如：100.0）
    birthday date,
    insert_time timestamp
);
```



#### 复制表

```sql
# 复制表
create table 表名 like 被复制的表名;
```







### 5.2 查询表

#### 查询所有表

```sql
# 查询某个数据库的所有表的名称
show tables;
```





#### 查询表结构

```sql
# 查询表结构
desc 表名;
```



#### 查询表字符集

#### 查询表创建语句

```sql
# 查询表字符集/查询表创建语句
show create table 表名;
```





### 5.3 修改表

#### 修改表名

```sql
# 修改表名
alter table 表名 rename to 新的表名;
```



#### 修改表的字符集

```sql
# 修改字符集为utf-8
alter table 表名 character set utf8;
```



#### 添加一列

```sql
# 添加列
alter table 表名 add 列名 数据类型；
```



#### 修改某一列

```sql
# 修改列名和数据类型
alter table 表名 change 列名 新列名 新数据类型;

# 修改数据类型
alter table 表名 modify 列名 新数据类型;
```



#### 删除某一列

```sql
# 删除列
alter table 表名 drop 列名;
```



### 5.4 删除表

```sql
# 删除表
drop table 表名;

# 判断是否存在，再删除
drop table if exists 表名;
```





## 6 操作数据

### 6.1 添加数据

语法：

```sql
# 插入数据
insert into 表名(列名1,列名2,...,列名n) values(值1,值2,...,值n);
```

注意列名和表名要一一对应，如果不定义列名，则给所有的列添加值：

如：

```sql
# 插入数据
insert into 表名 values(值1,值2,...,值n);
```



示例：

```sql
# 插入数据
insert into student(id,name)values(1,"小明");
```



### 6.2 删除数据

```sql
# 删除数据
delete from 表名 [where 条件]
```





示例：

```sql
# 删除id=1的数据
delete from student where id=1;

# 删除全部数据（不建议使用）
# 有多少条记录，就会进行多少次删除操作，效率比较低
delete from student;
```





**注意如果不加条件会删除所有记录，但如果要删除所有的数据也不建议使用该命令，因为它会一条一条的删除数据，效率比较低下，建议使用`TRUNCATE` ，它会先删除表，再创建一个一模一样的表。**

```sql
# 删除全部数据
# 删除表，再创建一个相同的表
truncate table 表名; 
```



### 6.3 修改数据

语法：

```sql
# 修改数据
update 表名 set 列名1 = 值1,列名2 = 值2,...[where 条件];
```



**注意：如果不加任何条件，会将表中所有的记录全部修改。**



### 6.4 查询数据

#### 语法

```sql
select
	字段列表
from
	表名列表
where
	条件列表
group by
	分组字段
having
	分组之后的条件
order by
	排序
limit
	分页限定
```



测试表：

```sql
create table student(
	id int,
    name varchar(32),
    age int,
    english double(4,1),
    math double(4,1)
)
```



#### 基础查询

1. 多个字段查询

   ```sql
   select 字段名1,字段名2,... from 表名;
   
   select * from 表名; -- 查询所有字段
   ```

2. 去除重复： 使用关键字 `distinct`

3. 计算列

   - 一般可以使用四则运算计算一些列的值（一般只会对数值型的数据进行计算）。

   - 如果有字段为null参与运算，结果都会null，这时可以使用`ifnull`关键字，具体格式如下

     ```sql
     ifnull(表达式1,表达式2)
     -- 表达式1：需要判断是否为null的字段
     -- 表达式2：如果该字段为空，所替换的值
     ```

4. 取别名

   使用`as`关键字，`as`也可以省略。

```sql
select * from student;

select age from student;

--去除重复结果
select distinct age from student;

--技术math和english分数之和
--如果有NULL参与，结果都为NULL
select name,math,english,math+english from student;

--技术math和english分数之和
--使用IFNULL
select name,math,english,math+IFNULL(english,0) from student;

--字段取别名
select name,math,enlish,math+IFNULL(endlish,0) as 总分 from student;
select name,math,enlish,math+IFNULL(endlish,0)  总分 from student; --可以不用as，直接空格也可以
```



#### 条件查询

使用 `where`字句

1. 使用运算符 `> , < , <= , >= , = ，!= , <>`

```sql
--查询年龄大于（等于）20岁
select * from student where age > 20;
select * from student where age >= 20;

--查询年龄等于20岁
select * from student where age = 20; --是一个等号，不是==

--查询年龄不等于20岁  != 和 <>效果是一样的
select * from student where age != 20;
select * from student where age <> 20;


```



2. 使用 `&&`  , `and`  ,`between ... and`

```sql
--查询年龄大于等于20 小于等于30
select * from student where age >=20 && age <= 30; --推荐使用 and
select * from student where age >=20 and age <= 30;
select * from student where age between 20 and 30;
```



3. 使用`in (集合)`

```sql
--查询年龄22岁，19岁，25岁的信息
select * from student where age = 22 or age = 19 or age = 25;
select * from student where age in (22,19,25);
```



4. 使用 `is null` ,`is not null`

```sql
--查询英语成绩为null
select * from student where english = null; -- 这是不对的写法，null不能使用 = 和！=判断
select * from student where english is null; -- 使用is关键字

--查询英语成绩不为null
select * from  student where english is not null;
```



5. 使用`like`

占位符：

- `_`  ： 单个任意字符
- `%` ： 多个任意字符

```sql
--查询name中姓马的有哪些？  使用like
select * from student where name like '马%';  --单引号 双引号都行

--查询name中第二个字是“华”的有哪些 ？
select * from student where name like "_华%";

--查询姓名是三个字的人
select * from student where name like '___';

--查询姓名中包含“马”的人
select * from student where name like '%马%';

```



#### 排序查询

语法：

```sql
-- order by 字句
order by 排序字段1 ,排序方式1, 排序字段2,排序方式2 ....   
```

排序方式：

- `ASC` ：升序，默认为升序。
- `DESC` ： 降序。

排序规则： 如果有多个排序条件，则当前面的条件值一样是，才会判断后面的条件值。

```sql
-- 按数学成绩排序
select * from student order by math asc; --升序
select * from student order by math desc; --降序

-- 先按数学成绩排序，如果数学成绩一样，再按英语成绩排序
select * from student order by math asc, english asc;
```



#### 聚合函数

将一列数据作为一个整体，进行纵向的计算。

> **注意：聚合函数会排除`null`的值**
>
> 解决方案：
>
> （1）选择不包含`null`的列进行计算
>
> （2）使用`ifnull`函数



1. `count` ：计算个数
   - 一般选择非null的列，如主键  。
   - 可以使用`count(*)`

```sql
select count(name) from student;
select count(ifnull(english ,0)) from student;
select count(id) from student;
select count(*) from student;
```



2. `max` ：计算最大值
3. `min` : 计算最小值
4. `sum` ：计算和
5. `avg` : 计算平均值



```sql
select max(math) from student;
select min(math) from student;
select sun(math) from student;
select avg(math) from student;
```





#### 分组查询

语法

```sql
-- group by
group by 分组字段;
```



> 注意：
>
> 1. 分组之后查询的字段只能是分组字段和聚合函数
> 2. `where`和`having`的区别？
>    - `where`在分组前进行限定，不满足条件不参与分组；`having`在分组后进行限定，如果不满足条件则不会被查出来。
>    - `where`后不可以跟聚合函数，`having`后面可以进行聚合函数判断。

```sql
-- 按照性别分组。分别查询男、女同学的数学平均分
select sex ,avg(math) from student group by sex;

-- 按照性别分组。分别查询男、女同学的数学平均分,人数
select sex ,avg(math),count(id) from student group by sex;

-- 按照性别分组。分别查询男、女同学的数学平均分,人数。要求分数小于70的人不参与分组
select sex ,avg(math),count(id) from student where math > 70 group by sex;

-- 按照性别分组。分别查询男、女同学的数学平均分,人数。要求分数小于70的人不参与分组,且分组后组的人数要大于2
select sex ,avg(math),count(id) from student where math > 70 group by sex having count(id);
```



#### 分页查询

语法：

```sql
-- limit
limit 开始的索引 ,每页查询的条数;
```

> 开始的索引 = （当前的页码 - 1）* 每页显示的条数

```sql
-- 每页显示3条记录
select * from student limit 0,3;  --第1页
select * from student limit 3,3;  --第2页
select * from student limit 6,3;  --第3页
```



> `limit`关键字是在`MySQL`中特有，`Oracle`和`SqlServer`没有



## 7 约束

概念： 对表中的数据进行限定，保证数据的正确性、有效性和完整性

分类：

1. 主键约束：` primary key`
2. 非空约束：`not null`
3. 唯一约束：`unique`
4. 外键约束：`foreign key`



### 7.1 非空约束

#### 创建表时添加非空约束



```sql
CREATE TABLE stu(
	id INT,
    name VARCHAR(20) NOT NULL --name为非空
);
```



#### 删除非空约束

```sql
-- 删除非空约束
alter table stu modify name varchar(20);
```



#### 创建表后添加非空约束

```sql
-- 添加非空约束
alter table stu modify name varchar(20) not null;
```



### 7.2 唯一约束

#### 创建表时添加唯一约束

```sql
create table stu(
	id int,
    phone_number varchar(20) unique -- 添加唯一约束
)
```

> 注意：MySQL中的唯一约束限定的列的值可以有多个null



#### 删除唯一约束

```sql
-- 删除唯一约束
-- alter table stu modify phone_number varchar(20);是删除不掉的。。
alter table stu drop index phone_number; 
```



####  创建表后添加唯一约束

```sql
-- 创建表后添加唯一约束
alter table stu modify phone_number varchar(20) unique;
```



### 7.3 主键约束

- 含义：非空且唯一
- 一张表只能有一个字段为主键
- 主键是表中记录的唯一标识



#### 创建表时添加主键

```sql
create table stu(
	id int primary key,
    name varchar(20)
)
```



#### 删除主键约束

```sql
-- 删除主键
alter table stu drop primary key;
```



#### 创建表后添加主键

```sql
-- 创建表后添加主键
alter table stu modify id int primary key;
```



#### **自动增长**

如果某一列是数值类型，使用`auto_increment` 可以来完成值的自动增长。

```sql
-- 创建表时添加自动增长
create table stu(
	id int primary key auto_increment,
    name varchar(20)
)

-- 删除自动增长
alter table stu modify id int;

-- 添加自动增长
alter talbe stu modify id int auto_increment;
```



### 7.4 键约束

`foreign key` 让表与表产生关系，从而保证数据的正确性



#### 创建表时添加外键

语法：

```sql
create table 表名(
	...
    外键列,
    constraint 外键名称 foreign key (外键列名称) references 主表名称(主表列名称)
);
```



示例：

```sql
create table department(
	id int primary key auto_increment,
    dep_name varchar(20),
    dep_location varchar(20)
);

create table employee(
	id int primary key auto_increment,
    name varchar(20),
    age int,
    dep_id int, -- 外键对于的主键
    constraint emp_dept_fk foreign key (dep_id) references department(id)
);
```



#### 删除外键

```sql
-- 删除外键
alter table employee drop foreign key emp_dept_fk;
```



#### 创建表后添加外键

```sql
-- 添加外键
alter table employee add  constraint emp_dept_fk foreign key (dep_id) references department(id);
```



#### 级联操作

- 级联更新： ON UPDATE CASCADE
- 级联删除：ON DELETE CASCADE

语法：

```sql
-- 级联更新和级联删除可以分别设置
ALTER TABLE 表名 ADD CONSTRAINT 外键名称 FOREIGN KEY (外键字段名称) ON UPDATE CASCADE ON DELETE CASCADE;
```



## 8 多表查询

>  笛卡尔积：有两个集合A、B，取这两个集合的所有组成情况。



### 8.1 内连接查询

#### 隐式内连接

使用`where`条件消除无用的数据

```sql
-- 查询所有员工信息和对应的部门信息
select * from emp,dept where emp.dept_id=dept.id;

-- 查询员工表的名称，性别，部门表的名称
select * from emp.name,emp.gender,dept.name from emp,dept where emp.dept.id = dept.id;a

-- 取别名
select 
	t1.name,  --员工表的姓名
	t1,gender, --员工表的性别
	t2.name --部门表的名称
from
	emp t1, dept t2
where
	t1.dept_id = t2.id;
```



#### 显示内连接

语法：

```sql
-- 显示内连接
select 字段名称 from 表名1 [inner] join 表名2 on 条件;
```





示例：

```sql
select * from emp inner join dept on emp.dept_id = dept.id;

-- inner是可以省略的
select * from emp join dept on emp.dept_id = dept.id;


```





### 8.2 外连接查询

#### 左外连接

语法：

```sql
-- outer可以不写
select * from 字段列表 from 表1 left [outer] join 表2 on 条件;
```



示例：

```sql
select
	t1.*,t2.name
from 
	emp t1 left join dept t2 on t1.dept_id = t2.id;
```



#### 右外连接

语法：

```sql
-- outer可以不写
select * from 字段列表 from 表1 right [outer] join 表2 on 条件;
```



示例：

```sql
select
	t1.*,t2.name
from 
	emp t1 left join dept t2 on t1.dept_id = t2.id;
```



### 8.3 子查询

查询中嵌套查询，成嵌套查询为子查询。



示例：

```sql
-- 查询工资最高的员工信息
-- 1 查询最高工资是多少  --->9000
select max(salary) from emp;
-- 2 查询员工信息，且工作等于9000
select * from emp where emp.salary = 9000;

--使用一条sql完成
select * from emp where emp.salary = (select max(salary) from emp);
```



#### 子查询的结果是单行单列的

子查询的结果可以作为条件，使用运算符（`> 、 >= 、 < 、 <=、 =`）去判断

```sql
-- 查询员工工资小于平均工资的人
select * from emp where emp.salary < (select avg(salary) from emp)
```



#### 子查询的结果是多行单列的

子查询可以作为条件，使用运算符`in`来判断



```sql
-- 查询 财务部 和 市场部 的所有员工信息
select id from dept where name = '财务部' or name = '市场部'; -- -->3 ，2
select * from emp where dept_id = 3 or dept_id = 2;


-- 使用子查询
select * from emp where dept_id in (select id from dept where name = '财务部' or name = '市场部');
```



#### 子查询的结果是多行多列的

子查询可以作为一张虚拟表参与查询

```sql
-- 查询员工的入职日期是 2011-11-11 之后的员工信息和部门信息
select 
	* 
from
	dept t1,
	(select * from emp where emp.join_date > '2011-11-11') t2
where 
	t1.id = t2.dept_id;
```







## 9 事务

### 9.1 事务的基本介绍

如果一个包含多个步骤的业务操作，被事务管理，那么这些操作要么同时成功，要么同时失效。

操作：

1. 开启事务： `start transacion;`
2. 回滚： `rollback;`
3. 提交：`commit;`

```sql
-- 张三给李四转账500元
-- 0 开启事务
start transaction

-- 1 张三账户 - 500
update acount set balance = balance -500 where name = "zhangsan"


出错了...

--2 李四账户 + 500
update acount set balance = balance + 500 where name = "lisi"

-- 发现执行没有问题，提交事务
commit;

-- 发现出问题了，回滚事务
rollback;
```

事务提交的两种方式：

1. 自动提交

   MySQL中的事务是自动提交的，一条DML（增删改）语句会自动提交一次事务

2. 手动提交

   Oracle数据库默认是手动提交事务，需要先开启事务，然后再提交

MySQL中修改事务默认提交方式：

```sql
-- 查看事务的默认提交方式
-- 1 代表自动提交 
-- 0 代表手动提交
SELECT @@autocommit;

-- 修改默认提交方式
set @@autocommit = 0

-- mysql中将提交方式改为手动提交后，需要执行COMMIT才会生效

```





### 9.2 事务的四大特征

1. 原子性：是不可分割的最小操作单位，要么同时成功，要么同时失败。
2. 持久性：当事务提交或者回滚吼，数据库会持久化的保存数据。
3. 隔离性：多个事务之间，相互独立。
4. 一致性：事务操作前后，数据总量不变。

### 9.3 事务的隔离级别

多个事务之间隔离的相互独立的。但是如果多个事务操作同一批数据，则会引发一些问题，设置不同的隔离级别就可以解决这些问题。



存在的问题：

1. 赃读： 一个数据读取到另一个事务中没有提交的数据。
2. 不可重复读（虚读）：在同一个事务中，两次读取到的数据不一样。
3. 幻读：一个事务操作（DML）数据表中所有的记录，另一个事务添加了一条数据，则地一个事务查询不到自己的修改。



隔离级别：

1. read uncommitted：读未提交

   产生的问题：赃读、不可重复读、幻读

2. read committed：读已提交（Oracle默认）

   产生的问题：不可重复读、幻读

3. repeatable read：可重复读（MySQL默认）

   产生的问题：幻读

4. serializable ：串行化

   可以解决所有的问题



> 注意：隔离级别从小到大安全性越来越高，但效率越来越低。

## 10用户管理和权限管理

### 10.1 用户管理

#### 查询用户

```sql
-- 1 切换到mysql数据库
use mysql;

-- 2 查询uesr表
select * from user;
```

> 通配符： `% `表示可以在任意主机使用用户登录数据库



#### 创建用户

语法：

```sql
-- 创建用户
create user '用户名'@'主机名' identified by '密码';
```

示例：

```sql
create user 'zhangsan'@'localhost' identified by '123';
create user 'lisi'@'%' identified by '123'; --任意主机都可以访问
```

#### 删除用户

```sql
--删除用户
drop user '用户名'@'主机名';
```



#### 修改用户密码

使用sql的方式修改：

```SQL
-- 修改密码
update user set password = password('新密码') where user = '用户名';

-- 示例
update user set password = password('abc') where user = 'zhangsan';
```

第二种方式：

```sql
-- 修改密码
set password for '用户名'@'主机名'=password('新密码');
```



#### MySQL中忘记了root用户密码？

1. 停止MySQL服务

   ```sql
   # windows下需要管理员权限
   net stop mysql
   ```

2. 使用无验证方式启动MySQL

   ```sql
   # 窗口会停留
   mysqld --skip-grant-tables
   ```

3. 登录MySQL修改密码

   ```sql
   # 重新打开一个窗口，直接输入mysql回车就可以登录数据库
   mysql
   ```

   

   修改密码：

   ```sql
   use mysql;
   update user set password = password('新密码') where user = 'root';
   ```

   

### 10.2 权限管理

#### 查询权限

```sql
-- 查询权限
SHOW GRANTS FOR '用户名'@'主机名';

-- 示例
SHOW GRANTS FOR 'root'@'%';
```



#### 授予权限

```sql
-- 授予权限
GRANT 权限列表 ON 数据库名.表名 TO '用户名'@'主机名';

--示例
GRANT SELECT,DELETE,UPDATE ON db3.acount TO 'lisi'@'%';

-- 授予所有权限，在任意数据库任意表上
GRANT ALL ON *.*  TO 'zhangsan'@'localhost';
```



#### 撤销权限

```sql
-- 撤销权限
REVOKE 权限列表 ON 数据库.表名 FROM '用户名'@'主机名';
```



## 11 数据库的备份和还原



### 11.1 命令行方式

#### 备份

使用命令：

```sql
-- 在命令行中执行
mysqldump -u用户名 -p密码 数据库名称  > 保存的路径
```



#### 还原

1. 登陆数据库

2. 创建数据库

3. 使用数据库

4. 执行备份文件

   ```SQL
   source 文件路径
   ```

   



### 11.2 图形化工具

点击备份导出导入即可。











## 12 补充：MySQL 数据类型

来源：https://www.runoob.com/mysql/mysql-data-types.html

MySQL中定义数据字段的类型对你数据库的优化是非常重要的。

MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。

------

### 12.1 数值类型

MySQL支持所有标准SQL数值数据类型。

这些类型包括严格数值数据类型（INTEGER、SMALLINT、DECIMAL和NUMERIC)，以及近似数值数据类型(FLOAT、REAL和DOUBLE PRECISION）。

关键字INT是INTEGER的同义词，关键字DEC是DECIMAL的同义词。

BIT数据类型保存位字段值，并且支持MyISAM、MEMORY、InnoDB和BDB表。

作为SQL标准的扩展，MySQL也支持整数类型TINYINT、MEDIUMINT和BIGINT。下面的表显示了需要的每个整数类型的存储和范围。

![](https://raw.githubusercontent.com/foochane/java-learning/master/image/数值类型.png)

------

### 12.2 日期和时间类型

表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。

每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

TIMESTAMP类型有专有的自动更新特性，将在后面描述。

![](https://raw.githubusercontent.com/foochane/java-learning/master/image/日期和时间类型.png)

------

### 12.3 字符串类型

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

![](https://raw.githubusercontent.com/foochane/java-learning/master/image/字符串类型.png)

CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。

有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。