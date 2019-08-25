# python with MySQL

- [python with MySQL](#python-with-mysql)
  - [Datatype & Constraint](#datatype--constraint)
  - [MySQL on Ubuntu](#mysql-on-ubuntu)
    - [navicat](#navicat)
    - [another host to connect MySQL](#another-host-to-connect-mysql)
    - [Operation](#operation)
      - [database operation](#database-operation)
      - [table operation](#table-operation)
      - [record operation](#record-operation)
    - [backup and recovery](#backup-and-recovery)
  - [query](#query)
    - [聚合](#%e8%81%9a%e5%90%88)
    - [分组](#%e5%88%86%e7%bb%84)
    - [排序](#%e6%8e%92%e5%ba%8f)
    - [分页](#%e5%88%86%e9%a1%b5)
    - [query summary](#query-summary)
  - [relationship](#relationship)
    - [外键(foreign key)](#%e5%a4%96%e9%94%aeforeign-key)
    - [外键的级联操作(作为了解，实际上没什么卵用)](#%e5%a4%96%e9%94%ae%e7%9a%84%e7%ba%a7%e8%81%94%e6%93%8d%e4%bd%9c%e4%bd%9c%e4%b8%ba%e4%ba%86%e8%a7%a3%e5%ae%9e%e9%99%85%e4%b8%8a%e6%b2%a1%e4%bb%80%e4%b9%88%e5%8d%b5%e7%94%a8)
    - [外键summary](#%e5%a4%96%e9%94%aesummary)
  - [join](#join)
  - [自关联查询](#%e8%87%aa%e5%85%b3%e8%81%94%e6%9f%a5%e8%af%a2)
  - [view](#view)
  - [Transaction](#transaction)
  - [Index](#index)
  - [final example](#final-example)
  - [常用内置函数](#%e5%b8%b8%e7%94%a8%e5%86%85%e7%bd%ae%e5%87%bd%e6%95%b0)
    - [字符串函数](#%e5%ad%97%e7%ac%a6%e4%b8%b2%e5%87%bd%e6%95%b0)
  - [MySQL with Python](#mysql-with-python)
    - [封装](#%e5%b0%81%e8%a3%85)
    - [python交互实例-用户登录](#python%e4%ba%a4%e4%ba%92%e5%ae%9e%e4%be%8b-%e7%94%a8%e6%88%b7%e7%99%bb%e5%bd%95)
  - [sqlalchemy](#sqlalchemy)

数据库:
- 目的: 存储数据
- 优点: 优化读写
- 大多DB是基于E-R关系模型, 不同的数据库的优化读写方式不同;

E-R模型三范式:后一个范式，都是在前一个范式的基础上建立的
- 第一范式（1NF)：列不可拆分
- 第二范式（2NF)：唯一标识
- 第三范式（3NF)：引用主键

数据库的数据操作:CRUD(C: Create, R: Retrive, U: Update, D: Delete)

SQLite: 轻量级的数据库; 读写数据比文本文件快；用于移动端;
> 数据量少怎么存取都无所谓；当数据量大了，就需要组织数据，更加有效的存取数据

数据库分类:
- 文档型: 如sqlite，就是一个文件，通过对文件的复制完成数据库的复制
- 服务型: 如mysql、postgre，数据存储在一个物理文件中，但是需要使用终端以tcp/ip协议连接，进行数据库的读写操作
  >![](res/database01.png)

## Datatype & Constraint

```bash
# 数字
decimal(5,2)# 1.2, 1.21,123.23
# 字符串
# 既然有文本，创建数据库的时候就要指定编码格式，
char(8) # 有限长度字符,不够会在右侧补充空格
varchar(8) #有限长度，不够不会补充空格; MySQL直接截取保存，其他数据库会报错;
text # 大文本

datetime # 日期
bit # 布尔
```

primary key只能有一个;unique可以有多个;

物理上的存储是按照primarykey来存储的，而不是按照unique来存储的;

## MySQL on Ubuntu

install mysql:
1. install apt [repo](https://dev.mysql.com/downloads/repo/apt/)(`sudo dpkg -i mysql-apt-config_xxx_all.deb`)
2. `sudo apt update`
3. `sudo apt install mysql-server mysql-client`(按提示输入mysql的root密码)
4. `systemctl enable mysql && systemctl start mysql`
5. install [workbench](https://dev.mysql.com/downloads/workbench/)

如果安装过程中不提示输入密码: 

```bash
# 查看默认密码
sudo vim /etc/mysql/debian.cnf
# 登录
mysql -u debian-sys-maint -p 

# 添加root账号
mysql> update mysql.user set authentication_string=password('xxxxxx') where user='root'and Host = 'localhost';
# 修改plugin
mysql> USE mysql;
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='root';
mysql> FLUSH PRIVILEGES;
mysql> exit;

# 用root登录
mysql -u root -p

# simple test
show databases;
use mysql;
show tables;
desc user;
select * from user \G;
```

config mysql

```bash
# 允许远程连接
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
# 将bind-address=127.0.0.1注释

# 登陆mysql
mysql -u root -p
# 登陆窗口运行命令
# 所有远程ip都可用13810455459访问root用户的所有数据库
# 远程登陆的时候用13810455459登陆，而不是mysql的root密码
# 
# grant all privileges on *.* to 'root'@'%' identified by '13810455459' with grant option;
grant all privileges on *.* to 'grey'@'%' identified by '13810455459' with grant option;
# show grants for grey;
flush privileges;
exit

# 重启服务
systemctl restart mysql
# 查看是否运行
ps -ef|grep mysql
```

Steps:
1. 下载[MySQL5.7 zip](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)
2. 解压上面zip重命名为Engine, 然后同一目录新建Data文件夹
3. C:\Windows下创建my.ini
4. 进入Engine文件夹`mysqld --initialize`
5. 进入Data文件夹下，复制`*.err`文件中的root账号密码

```ini
; C:\Windows\my.ini
[mysqld]
innodb_flush_method=normal
basedir='D:\MySQL\Engine'
datadir='D:\MySQL\Data'
```

```bash
# in window
# 按照timonthy安装mysql之后
# 启动
mysqld.exe --console
# 登录mysql
mysql.exe -u root -p
# 给予权限, *.*表示所有数据库; test.*表示test数据库下的所有tables;
grant all privileges on *.* to 'root'@'%' identified by '13810455459' with grant option;
flush privileges;
exit
# 然后就可以用navicat连接了
```

```bash
# MySQL8之后的变化
CREATE USER `root`@`%` IDENTIFIED BY '13810455459';
GRANT ALL ON *.* TO `root`@`%` WITH GRANT OPTION;
```

这样就可以用navicat在widnows上面访问mysql的数据库了

### navicat

![](res/navicat01.png)

连接完成之后，会出现四个系统级数据库:

- information_schema
- mysql
- performance_schema
- sys

New Database:

![](res/navicat02.png)

根据E-R模型设计得到的E就是一个Table;

一般设计table的时候需要预留field，防止别人改要求; 不要删除field(不用它),换成其他的field的就行

![](res/navicat03.png)

如果删除数据，对应的id是不会变的，为了降低维护成本(不用重新index)

![](res/navicat04.png)

重要的数据要做逻辑删除:创建一个isDelete字段(DeleteFlag, DeleteStatus),但是物理的数据不能删除; navicat选中del是物理删除;

![](res/navicat05.png)

### another host to connect MySQL

```bash
# client connect to server
mysql -h 192.168.128.136 -u root -p

# mysql
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.21    |
+-----------+

mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2018-04-05 11:54:20 |
+---------------------+
```

### Operation

`sql`的关键字不区分大小写(因为是类B语言)

#### database operation

```sql
-- show all databases;
show databases;

create database teachers charset=utf8;

use teachers;

-- see current selected database
select database();

drop database teachers;
```
#### table operation

```sql
-- use world database
use python3;

show tables;

desc students;

-- rename table name
rename table studens to new_students;

drop table new_students;
```

```bash
# create a table
mysql> create table students(
    -> id int auto_increment primary key not null,
    -> name varchar(10) not null,
    -> gender bit default 1,
    -> birthday datetime);

# show table structure
mysql> desc students;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(10) | NO   |     | NULL    |                |
| gender   | bit(1)      | YES  |     | b'1'    |                |
| birthday | datetime    | YES  |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+

# 修改table主要是修改表的结构，add、drop、change 列名 列属性;
# 不能修改列名
mysql> alter table students add isDelete bit default 0;

# show table structure
mysql> desc students;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(10) | NO   |     | NULL    |                |
| gender   | bit(1)      | YES  |     | b'1'    |                |
| birthday | datetime    | YES  |     | NULL    |                |
| isDelete | bit(1)      | YES  |     | b'0'    |                |
+----------+-------------+------+-----+---------+----------------+

# 查看表的创建语句
mysql> show create table students;

| Table    | Create Table                                                         
| students | CREATE TABLE `students` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL,
  `gender` bit(1) DEFAULT b'1',
  `birthday` datetime DEFAULT NULL,
  `isDelete` bit(1) DEFAULT b'0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
# engine不一样, 底层的结构就会不一样
# charset默认继承自database
```

#### record operation

```bash
mysql>use myDB;

# 增加数据
# 第一个0是为了占位置
mysql> insert into students values(0,'黄蓉','1993-3-29',0);

# 查询数据
mysql> select * from students;
+----+-----------+------------+--------+
| id | name      | birthday   | gender |
+----+-----------+------------+--------+
|  1 | 郭靖      | 1990-02-12 |       |
|  3 | 曹操      | 2018-03-21 |       |
|  4 | 诸葛亮    | 2019-06-09 |       |
|  5 | 司马懿    | 2018-08-09 |       |
|  6 | 黄蓉      | 1993-03-29 |        |
+----+-----------+------------+--------+

# 第二种插入数据的方式，部分字段(而且是not null), 顺序无所谓
mysql> insert into students(name,birthday) values('孙权','1996-1-21');
mysql> insert into students(birthday, name) values('1993-10-20','杨过');

mysql> select * from students;
+----+-----------+------------+--------+
| id | name      | birthday   | gender |
+----+-----------+------------+--------+
|  1 | 郭靖      | 1990-02-12 |       |
|  3 | 曹操      | 2018-03-21 |       |
|  4 | 诸葛亮    | 2019-06-09 |       |
|  5 | 司马懿    | 2018-08-09 |       |
|  6 | 黄蓉      | 1993-03-29 |        |
|  7 | 孙权      | 1996-01-21 |       |
+----+-----------+------------+--------+

# 插入多条数据
mysql> insert into students values(0,'小龙女','1996-1-22',0),(0,'郭湘','1998-2-22',0);
mysql> insert into students(name,birthday) values('周瑜','2000-1-22'),('诸葛瑾','1996-2-23');
mysql> select * from students;
+----+-----------+------------+--------+
| id | name      | birthday   | gender |
+----+-----------+------------+--------+
|  1 | 郭靖      | 1990-02-12 |       |
|  3 | 曹操      | 2018-03-21 |       |
|  4 | 诸葛亮    | 2019-06-09 |       |
|  5 | 司马懿    | 2018-08-09 |       |
|  6 | 黄蓉      | 1993-03-29 |        |
|  7 | 孙权      | 1996-01-21 |       |
|  8 | 小龙女    | 1996-01-22 |        |
|  9 | 郭湘      | 1998-02-22 |        |
| 10 | 周瑜      | 2000-01-22 |       |
| 11 | 诸葛瑾    | 1996-02-23 |       |
+----+-----------+------------+--------+
```

```bash
# 修改数据
mysql> select * from students;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       | NULL     |
|  3 | 曹操      | 2018-03-21 |       | NULL     |
|  4 | 诸葛亮    | 2019-06-09 |       | NULL     |
|  5 | 司马懿    | 2018-08-09 |       | NULL     |
|  6 | 黄蓉      | 1993-03-29 |        | NULL     |
|  7 | 孙权      | 1996-01-21 |       | NULL     |
|  8 | 小龙女    | 1996-01-22 |        | NULL     |
|  9 | 郭湘      | 1998-02-22 |        | NULL     |
| 10 | 周瑜      | 2000-01-22 |       | NULL     |
| 11 | 诸葛瑾    | 1996-02-23 |       | NULL     |
+----+-----------+------------+--------+----------+

mysql> update students set birthday='1999-1-1' where id=3;

mysql> select * from students;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       | NULL     |
|  3 | 曹操      | 1999-01-01 |       | NULL     |
|  4 | 诸葛亮    | 2019-06-09 |       | NULL     |
|  5 | 司马懿    | 2018-08-09 |       | NULL     |
|  6 | 黄蓉      | 1993-03-29 |        | NULL     |
|  7 | 孙权      | 1996-01-21 |       | NULL     |
|  8 | 小龙女    | 1996-01-22 |        | NULL     |
|  9 | 郭湘      | 1998-02-22 |        | NULL     |
| 10 | 周瑜      | 2000-01-22 |       | NULL     |
| 11 | 诸葛瑾    | 1996-02-23 |       | NULL     |
+----+-----------+------------+--------+----------+

# 如果没有where, 所有record都改了
mysql> update students set birthday='2001-1-1',name='孔明' where name='诸葛亮';

mysql> select * from students;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       | NULL     |
|  3 | 曹操      | 1999-01-01 |       | NULL     |
|  4 | 孔明      | 2001-01-01 |       | NULL     |
|  5 | 司马懿    | 2018-08-09 |       | NULL     |
|  6 | 黄蓉      | 1993-03-29 |        | NULL     |
|  7 | 孙权      | 1996-01-21 |       | NULL     |
|  8 | 小龙女    | 1996-01-22 |        | NULL     |
|  9 | 郭湘      | 1998-02-22 |        | NULL     |
| 10 | 周瑜      | 2000-01-22 |       | NULL     |
| 11 | 诸葛瑾    | 1996-02-23 |       | NULL     |
+----+-----------+------------+--------+----------+
```

```bash
mysql> select * from students;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  6 | 黄蓉      | 1993-03-29 |        |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
|  9 | 郭湘      | 1998-02-22 |        | NULL     |
| 10 | 周瑜      | 2000-01-22 |       |          |
| 11 | 诸葛瑾    | 1996-02-23 |       |          |
+----+-----------+------------+--------+----------+

# 物理删除操作
mysql> delete from students where id=9;
# 逻辑删除, 本质是修改
mysql> update students set isDelete=1 where id=11;

mysql> select * from students where isDelete=0;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  6 | 黄蓉      | 1993-03-29 |        |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
| 10 | 周瑜      | 2000-01-22 |       |          |
+----+-----------+------------+--------+----------+
```

### backup and recovery

把数据从一个服务器移动到另一个服务器;

```bash
su
cd /var/lib/mysql

mysqldump -u root -p myDB > /home/grey/myDB_bak.sql

# 在navicat中把myDB数据库删除
# 上面的sql文件中没有创建database
# 需要先创建database, 然后进行恢复
mysql> create database myAnotherDB charset=utf8;

mysql> exit

grey@grey-pc:~/Desktop$ mysql -u root -p myAnotherDB < /home/grey/myDB_bak.sql
```

## query

```bash
mysql> select gender from students;
+--------+
| gender |
+--------+
|       |
|       |
|       |
|       |
|        |
|       |
|        |
|       |
|       |
+--------+

# 消除重复行
mysql> select distinct gender from students;
+--------+
| gender |
+--------+
|       |
|        |
+--------+

mysql> select distinct gender,id from students;
+--------+----+
| gender | id |
+--------+----+
|       |  1 |
|       |  3 |
|       |  4 |
|       |  5 |
|        |  6 |
|       |  7 |
|        |  8 |
|       | 10 |
|       | 11 |
+--------+----+

mysql> select id,name from students where id <=5;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | 郭靖      |
|  3 | 曹操      |
|  4 | 孔明      |
|  5 | 司马懿    |
+----+-----------+

mysql> select id,name from students where id >5 and gender=1;
+----+-----------+
| id | name      |
+----+-----------+
|  7 | 孙权      |
| 10 | 周瑜      |
| 11 | 诸葛瑾    |
+----+-----------+

# 模糊查询
# like
# %表示任意多个任意字符
# _表示一个任意字符

mysql> select id,name from students where name like '_马_';
+----+-----------+
| id | name      |
+----+-----------+
|  5 | 司马懿    |
+----+-----------+

mysql> select id,name from students where name like '司%';
+----+-----------+
| id | name      |
+----+-----------+
|  5 | 司马懿    |
+----+-----------+

mysql> select id,name from students where name like '司%' or name like '%靖';
+----+-----------+
| id | name      |
+----+-----------+
|  1 | 郭靖      |
|  5 | 司马懿    |
+----+-----------+

# 范围查询
# in 不连续范围查询

mysql> select * from students where id in (1,3,8);
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
+----+-----------+------------+--------+----------+

# 连续范围查询
mysql> select * from students where id between 3 and 8;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  6 | 黄蓉      | 1993-03-29 |        |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
+----+-----------+------------+--------+----------+

# 先弄一个and与between构成一组
mysql> select * from students where id between 3 and 8 and gender=1;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  7 | 孙权      | 1996-01-21 |       |          |
+----+-----------+------------+--------+----------+
```

空判断条件:

```bash
# null类似python中的None,与''是不同的

select * from students where hometown is null;

select * from students where hometown is not null;

select * from students where hometown is not null and gender=0;
```

优先级:

- `()`，`not`，比较运算符，逻辑运算符
- `and`比`or`先运算，如果同时出现并希望先算or，需要结合`()`使用

### 聚合

为了快速统计数据，提供了5个聚合函数; 只能看到统计的结果, 看不到原始的数据集;

```bash
# count()
mysql> select count(*) from students;
+----------+
| count(*) |
+----------+
|        9 |
+----------+

# max()
mysql> select max(id) from students where gender=0;
+---------+
| max(id) |
+---------+
|       8 |
+---------+

# min()
mysql> select min(id) from students where gender=0;
+---------+
| min(id) |
+---------+
|       6 |
+---------+

# sum()
mysql> select sum(id) from students where gender=0 and isDelete=0;
+---------+
| sum(id) |
+---------+
|      14 |
+---------+

# avg()
mysql> select avg(id) from students where gender=0;
+---------+
| avg(id) |
+---------+
|  7.0000 |
+---------+

# 组合使用， 子查询
mysql> select * from students where id=(select max(id) from students where gender=0);
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  8 | 小龙女    | 1996-01-22 |        |          |
+----+-----------+------------+--------+----------+

# 查询各学生的语文、数学、英语的成绩
mysql> select `name`,
    -> (select sco.score from scores sco inner join subjects sub on sco.sub_id=sub.id where sub.name='chinese' and stu_id=stu.id) as 语文,
    -> (select sco.score from  scores sco inner join subjects sub on sco.sub_id=sub.id where sub.name='math' and stu_id=stu.id) as 数学,
    -> (select sco.score from  scores sco inner join subjects sub on sco.sub_id=sub.id where sub.name='english' and stu_id=stu.id) as 英语
    -> from students stu;
+-----------+--------+--------+--------+
| name      | 语文   | 数学   | 英语   |
+-----------+--------+--------+--------+
| 郭靖      |   99.0 |  100.0 |   98.0 |
| 曹操      |   89.9 |   68.6 |   NULL |
| 孔明      |   NULL |   NULL |   NULL |
| 司马懿    |   NULL |   NULL |   NULL |
| 黄蓉      |   NULL |   NULL |   NULL |
| 孙权      |   NULL |   NULL |   NULL |
| 小龙女    |   NULL |   NULL |   NULL |
| 周瑜      |   NULL |   NULL |   NULL |
| 诸葛瑾    |   NULL |   NULL |   NULL |
+-----------+--------+--------+--------+

# or
mysql> select `name`,
    -> (select score from scores inner join subjects on scores.sub_id=subjects.id where subjects.name='chinese' and stu_id=stu.id) as 语文,
    -> (select score from  scores inner join subjects on scores.sub_id=subjects.id where subjects.name='math' and stu_id=stu.id) as 数学,
    -> (select score from  scores inner join subjects on scores.sub_id=subjects.id where subjects.name='english' and stu_id=stu.id) as 英语
    -> from students as stu;
+-----------+--------+--------+--------+
| name      | 语文   | 数学   | 英语   |
+-----------+--------+--------+--------+
| 郭靖      |   99.0 |  100.0 |   98.0 |
| 曹操      |   89.9 |   68.6 |   NULL |
| 孔明      |   NULL |   NULL |   NULL |
| 司马懿    |   NULL |   NULL |   NULL |
| 黄蓉      |   NULL |   NULL |   NULL |
| 孙权      |   NULL |   NULL |   NULL |
| 小龙女    |   NULL |   NULL |   NULL |
| 周瑜      |   NULL |   NULL |   NULL |
| 诸葛瑾    |   NULL |   NULL |   NULL |
+-----------+--------+--------+--------+
```

### 分组

分组的目的还是为了聚合; 如果没有group，对于每一组都要进行如上的一次查询，效率低;

按照一定的条件进行分组(数据相同的放到同一组中)，然后对某一组的record进行聚合;

```bash
# 非分组字段不能出现在select的后面
# select 列1,列2,聚合... from 表名 group by 列1,列2,列3...

mysql> select gender,count(*) from students group by gender;
+--------+----------+
| gender | count(*) |
+--------+----------+
|        |        2 |
|       |        7 |
+--------+----------+

mysql> select gender as '性别',count(*) as '总数' from students group by gender;
+--------+--------+
| 性别   | 总数   |
+--------+--------+
|        |      2 |
|       |      7 |
+--------+--------+
```

分组数据筛选;

- `where`: 是对原始数据集(`from`后面的)进行筛选;
- `having`: 是对分组结果集进行筛选;

![](res/having01.png)

```bash
# having后面的列必须是select后面的子集
# select 列1,列2,聚合... from 表名 group by 列1,列2,列3... having 列1,...聚合...

mysql> select gender as '性别',count(*) as '总数' from students group by gender having gender=0;
+--------+--------+
| 性别   | 总数   |
+--------+--------+
|        |      2 |
+--------+--------+

mysql> select gender as '性别',count(*) as '总数' from students group by gender having gender=0 and count(*)>1;
+--------+--------+
| 性别   | 总数   |
+--------+--------+
|        |      2 |
+--------+--------+

# 中文别名在这里不好使
mysql> select gender as '性别',count(*) as 'number' from students group by gender having gender=0 and number>1;
+--------+--------+
| 性别   | number |
+--------+--------+
|        |      2 |
+--------+--------+
```

### 排序

```bash
# 默认是 asc
# select * from 表名 order by 列1 asc|desc,列2 asc|desc,...

mysql> select * from students where isDelete=0 and gender=1 order by id desc;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
| 10 | 周瑜      | 2000-01-22 |       |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  1 | 郭靖      | 1990-02-12 |       |          |
+----+-----------+------------+--------+----------+

mysql> update subjects set isDelete=1 where name='math' or name='english';
mysql> update subjects set isDelete=1 where name in ('biology','physics');
mysql> select * from subjects;
+----+-------------+----------+
| id | name        | isDelete |
+----+-------------+----------+
|  1 | math        |         |
|  2 | chinese     |          |
|  3 | english     |         |
|  4 | programming |          |
|  5 | physics     |         |
|  6 | chemistry   |          |
|  7 | biology     |         |
+----+-------------+----------+

mysql> select * from subjects where isDelete=1 order by name desc;
+----+---------+----------+
| id | name    | isDelete |
+----+---------+----------+
|  5 | physics |         |
|  1 | math    |         |
|  3 | english |         |
|  7 | biology |         |
+----+---------+----------+
```

### 分页

常常配合order by来使用

```bash
# limit
# start用的不是id, 而是默认的编号index=0,1,2,3....
# select * from 表名 limit start,count

mysql> select * from subjects limit 0,6;
+----+-------------+----------+
| id | name        | isDelete |
+----+-------------+----------+
|  1 | math        |         |
|  2 | chinese     |          |
|  3 | english     |         |
|  4 | programming |          |
|  5 | physics     |         |
|  6 | chemistry   |          |
+----+-------------+----------+

mysql> select * from subjects limit 1,6;
+----+-------------+----------+
| id | name        | isDelete |
+----+-------------+----------+
|  2 | chinese     |          |
|  3 | english     |         |
|  4 | programming |          |
|  5 | physics     |         |
|  6 | chemistry   |          |
|  7 | biology     |         |
+----+-------------+----------+

# 每页显示m条数据，当前显示第n页(n从1开始)，但是index从0开始
select * from students where isdelete=0 limit (n-1)*m,m
```

example: `OFFSET`必须配合`LIMIT`使用

```sql
SELECT Name, Population FROM city LIMIT 8
SELECT Name, Population FROM city LIMIT 5 OFFSET 3
SELECT Name, Population FROM city LIMIT 5, 3
```

```bash
# result1
+----------------+------------+
| Name           | Population |
+----------------+------------+
| Kabul          |    1780000 |
| Qandahar       |     237500 |
| Herat          |     186800 |
| Mazar-e-Sharif |     127800 |
| Amsterdam      |     731200 |
| Rotterdam      |     593321 |
| Haag           |     440900 |
| Utrecht        |     234323 |
+----------------+------------+
# result2: 3~8
+----------------+------------+
| Name           | Population |
+----------------+------------+
| Mazar-e-Sharif |     127800 |
| Amsterdam      |     731200 |
| Rotterdam      |     593321 |
| Haag           |     440900 |
| Utrecht        |     234323 |
+----------------+------------+
# result3: 5~8
+-----------+------------+
| Name      | Population |
+-----------+------------+
| Rotterdam |     593321 |
| Haag      |     440900 |
| Utrecht   |     234323 |
+-----------+------------+
```

### query summary

完整的select语句:

```sql
select distinct 列*
from 表1 inner|left|right join 表2 on 表1与表2关系
where 关系表达式|逻辑表达式|范围|模糊|空判断
group by ... having ...
order by ... ASC|DESC
limit star,count

-- 执行顺序为：
-- from 表名
-- where ....
-- group by ...
-- select distinct *
-- having ...
-- order by ...
-- limit star,count
```

## relationship

- 实体与实体之间有3种对应关系，这些关系也需要存储下来
- 在开发中需要对存储的数据进行一些处理，用到内置的一些函数
- 视图用于完成查询语句的封装
- 事务可以保证复杂的增删改操作有效
- 但数据量巨大时, 为了提高查询速度可以通过索引来实现

实体间的三种对应关系, 关系的存储:

- 1v1: 关系字段存储在两个中任意一个都可以; 一般用于优化处理(有些数据查询用的多，有些数据的查询用的少，考虑设计成两张表,并且是一对一的关系)
- 1vn: 关系字段存储在n中;
- nvn: 新建一个table来存储关系

for example:

- 学生表和分数表是一对多的关系(一个学生的record对应多个分数record, 一个分数的record对应一个学生record)，那么在分数表存储关系
- 学生表和科目表是多对多的关系(一个学生的record对应多个科目record, 一个科目record对应多个学生record)，需要新建一个table来存储关系;
- 科目表和分数表示一对多的关系(一个科目的record对应多个分数的record, 一个分数的record对应一个科目的record)

关系设计的时候，实体直接的关系不能闭合，会有冗余数据；打开闭合，需要看访问的频率来打开；

### 外键(foreign key)

建立关系字段，对应于三范式的第三个: 引用primary key

![](res/relationship01.png)

上图中`stu_id`,`sub_id`的范围是由`students`表和`subjects`表来的，需要用到**外键约束**

**外键约束**: 建立好关系字段之后，为了保证关系字段的范围，而采用的约束(外键是外键，关系是关系); E-R图中存在关系，不存在外键; 等到了实际的表结构的时候才添加外键；

外键约束只是一个约束，关系在就行了；
> 所以使用`join`不需要建立foreign key

出现`1215 - Cannot add foreign key constraint`:

- 外键对应的字段数据类型不一致
- 设置外键时“删除时”设置为“SET NULL”
- 两张表的存储引擎不一致


```sql
-- method1
CREATE TABLE scores(
id int PRIMARY KEY auto_increment,
stu_id int,
sub_id int,
score DECIMAL(4,1)
);

alter table scores add constraint stu_sco foreign key(stu_id) references students(id);
-- 提供名字stu_sco的原因是方便以后删除constraint
-- 要先删constraint再删field
```

```bash
# method2
mysql> create table scores(
    -> id int primary key auto_increment,
    -> stu_id int(10) unsigned,
    -> sub_id int(11) unsigned,
    -> score decimal(4,1),
    -> foreign key(stu_id) references students(id),
    -> foreign key(sub_id) references subjects(id)
    -> );

mysql> desc scores;
+--------+------------------+------+-----+---------+----------------+
| Field  | Type             | Null | Key | Default | Extra          |
+--------+------------------+------+-----+---------+----------------+
| id     | int(11)          | NO   | PRI | NULL    | auto_increment |
| stu_id | int(10) unsigned | YES  | MUL | NULL    |                |
| sub_id | int(11) unsigned | YES  | MUL | NULL    |                |
| score  | decimal(4,1)     | YES  |     | NULL    |                |
+--------+------------------+------+-----+---------+----------------+
```

### 外键的级联操作(作为了解，实际上没什么卵用)

在删除students表的数据时，如果这个id值在scores中已经存在，则会存在**外键的级联操作**的问题

级联操作的类型包括：

- restrict（限制）：默认值，抛异常
- cascade（级联）：如果主表的记录删掉，则从表中相关联的记录都将被删除
- set null：将外键设置为空
- no action：什么都不做

推荐策略：不要使用物理删除，而要使用逻辑删除

### 外键summary

外键约束只是作为一个约束，实际上关系存在就行

## join

性能比子查询低，但是代码比子查询工整，推荐使用连接查询；

```bash
# 希望如下的查询结果
郭靖 math 100
```

```bash
mysql> select students.name,subjects.name as sub, scores.score 
    -> from scores
    -> inner join students on scores.stu_id=students.id
    -> inner join subjects on scores.sub_id=subjects.id;
+--------+---------+-------+
| name   | sub     | score |
+--------+---------+-------+
| 郭靖   | math    | 100.0 |
| 郭靖   | chinese |  99.0 |
| 郭靖   | english |  98.0 |
| 曹操   | math    |  68.6 |
| 曹操   | chinese |  89.9 |
+--------+---------+-------+
# 上面scores不可能冗余
```

```sql
-- 而且写法不唯一, 和上面同样的效果
select students.`name`,subjects.`name` as sub, scores.score
from students
inner join scores on scores.stu_id=students.id
inner join subjects on scores.sub_id=subjects.id;
```

![](res/join01.png)

```bash
# inner join
mysql> select * from students inner join scores on students.id=scores.stu_id;
+----+--------+------------+--------+----------+----+--------+--------+-------+
| id | name   | birthday   | gender | isDelete | id | stu_id | sub_id | score |
+----+--------+------------+--------+----------+----+--------+--------+-------+
|  1 | 郭靖   | 1990-02-12 |       |          |  1 |      1 |      1 | 100.0 |
|  1 | 郭靖   | 1990-02-12 |       |          |  2 |      1 |      2 |  99.0 |
|  1 | 郭靖   | 1990-02-12 |       |          |  3 |      1 |      3 |  98.0 |
|  3 | 曹操   | 1999-01-01 |       |          |  4 |      3 |      1 |  68.6 |
|  3 | 曹操   | 1999-01-01 |       |          |  5 |      3 |      2 |  89.9 |
+----+--------+------------+--------+----------+----+--------+--------+-------+

# left join
mysql> select *
    -> from students
    -> left join scores on students.id=scores.stu_id;
+----+-----------+------------+--------+----------+------+--------+--------+-------+
| id | name      | birthday   | gender | isDelete | id   | stu_id | sub_id | score |
+----+-----------+------------+--------+----------+------+--------+--------+-------+
|  1 | 郭靖      | 1990-02-12 |       |          |    1 |      1 |      1 | 100.0 |
|  1 | 郭靖      | 1990-02-12 |       |          |    2 |      1 |      2 |  99.0 |
|  1 | 郭靖      | 1990-02-12 |       |          |    3 |      1 |      3 |  98.0 |
|  3 | 曹操      | 1999-01-01 |       |          |    4 |      3 |      1 |  68.6 |
|  3 | 曹操      | 1999-01-01 |       |          |    5 |      3 |      2 |  89.9 |
|  4 | 孔明      | 2001-01-01 |       |          | NULL |   NULL |   NULL |  NULL |
|  5 | 司马懿    | 2018-08-09 |       |          | NULL |   NULL |   NULL |  NULL |
|  6 | 黄蓉      | 1993-03-29 |        |          | NULL |   NULL |   NULL |  NULL |
|  7 | 孙权      | 1996-01-21 |       |          | NULL |   NULL |   NULL |  NULL |
|  8 | 小龙女    | 1996-01-22 |        |          | NULL |   NULL |   NULL |  NULL |
| 10 | 周瑜      | 2000-01-22 |       |          | NULL |   NULL |   NULL |  NULL |
| 11 | 诸葛瑾    | 1996-02-23 |       |         | NULL |   NULL |   NULL |  NULL |
+----+-----------+------------+--------+----------+------+--------+--------+-------+

# right join
mysql> select * from students right join scores on students.id=scores.stu_id;
+------+--------+------------+--------+----------+----+--------+--------+-------+
| id   | name   | birthday   | gender | isDelete | id | stu_id | sub_id | score |
+------+--------+------------+--------+----------+----+--------+--------+-------+
|    1 | 郭靖   | 1990-02-12 |       |          |  1 |      1 |      1 | 100.0 |
|    1 | 郭靖   | 1990-02-12 |       |          |  2 |      1 |      2 |  99.0 |
|    1 | 郭靖   | 1990-02-12 |       |          |  3 |      1 |      3 |  98.0 |
|    3 | 曹操   | 1999-01-01 |       |          |  4 |      3 |      1 |  68.6 |
|    3 | 曹操   | 1999-01-01 |       |          |  5 |      3 |      2 |  89.9 |
+------+--------+------------+--------+----------+----+--------+--------+-------+

# mysql不支持full join, outer join;采用下面的方式实现
# emulate full join
SELECT * FROM students
LEFT JOIN scores ON students.id=scores.stu_id;
UNION
SELECT * FROM students
RIGHT JOIN scores ON students.id=scores.stu_id;
```

```bash
# 希望实现 姓名，平均分
mysql> select stu_id,avg(score)
    -> from scores
    -> group by stu_id;
+--------+------------+
| stu_id | avg(score) |
+--------+------------+
|      1 |   99.00000 |
|      3 |   79.25000 |
+--------+------------+

# join学生表，换掉stu_id
mysql> select students.name, avg(scores.score)
    # 下面这一句构成一个大的数据集，然后再执行其他操作
    -> from scores inner join students on scores.stu_id=students.id
    -> GROUP BY students.name;
+--------+-------------------+
| name   | avg(scores.score) |
+--------+-------------------+
| 曹操   |          79.25000 |
| 郭靖   |          99.00000 |
+--------+-------------------+

# 加上排序
mysql> select students.name, avg(scores.score) as ave_score
    -> from scores inner join students on scores.stu_id=students.id
    -> GROUP BY students.name
    -> ORDER BY ave_score DESC;
+--------+-----------+
| name   | ave_score |
+--------+-----------+
| 郭靖   |  99.00000 |
| 曹操   |  79.25000 |
+--------+-----------+

# 男生姓名、总分
mysql> select students.name, sum(scores.score) as sum_score
    -> from scores inner join students on scores.stu_id=students.id
    -> where students.gender=1
    # -> GROUP BY students.name
    # 要用学生id分组，因为存在同名的情况
    -> GROUP BY students.id
    -> ORDER BY sum_score DESC;
+--------+-----------+
| name   | sum_score |
+--------+-----------+
| 郭靖   |     297.0 |
| 曹操   |     158.5 |
+--------+-----------+

# 科目、平均分、最高分、最低分
mysql> select subjects.name as sub, avg(scores.score) as avg_score, max(scores.score), min(scores.score)
    -> from scores inner join subjects on scores.sub_id=subjects.id
    -> GROUP BY subjects.name
    -> ORDER BY avg_score DESC;
+---------+-----------+-------------------+-------------------+
| sub     | avg_score | max(scores.score) | min(scores.score) |
+---------+-----------+-------------------+-------------------+
| english |  98.00000 |              98.0 |              98.0 |
| chinese |  94.45000 |              99.0 |              89.9 |
| math    |  84.30000 |             100.0 |              68.6 |
+---------+-----------+-------------------+-------------------+
```

## 自关联查询

物理上是一张表，逻辑上是多张表(用别名区分)；

![](res/self_relation01.png)

对于数据库而言，创建一个table的开销很大；所以对于中国的省、市、区(县)[数据不过百万]可以放到一个表中; 这张表可以当作省、市、区来使用; 物理上和逻辑上是不同的;

而且上面的三个结构都是一样的(int, varchar, int)，完全可以放到同一个table中;

```bash
mysql> create table booktest_areas(
    -> id int primary key auto_increment not null,
    -> name varchar(20),
    -> parent_id int,
    -> foreign key(parent_id) references booktest_areas(id)
    -> );
mysql> desc booktest_areas;
+-----------+-------------+------+-----+---------+----------------+
| Field     | Type        | Null | Key | Default | Extra          |
+-----------+-------------+------+-----+---------+----------------+
| id        | int(11)     | NO   | PRI | NULL    | auto_increment |
| name      | varchar(20) | YES  |     | NULL    |                |
| parent_id | int(11)     | YES  | MUL | NULL    |                |
+-----------+-------------+------+-----+---------+----------------+

# import
source areas.sql

# 查看省的个数, 数据库有问题
mysql> select count(*) from booktest_areas where parent_id is NULL;
+----------+
| count(*) |
+----------+
|       33 |
+----------+

# 嵌套子查询用法太麻烦
mysql> select *
    -> from booktest_areas
    # 子查询只是为了获得省的id
    -> where parent_id=(select id from booktest_areas where name='湖北省');
+--------+--------------------------------+-----------+
| id     | name                           | parent_id |
+--------+--------------------------------+-----------+
| 420100 | 武汉市                         |    420000 |
| 420200 | 黄石市                         |    420000 |
| 420300 | 十堰市                         |    420000 |
| 420500 | 宜昌市                         |    420000 |
| 420600 | 襄阳市                         |    420000 |
| 420700 | 鄂州市                         |    420000 |
| 420800 | 荆门市                         |    420000 |
| 420900 | 孝感市                         |    420000 |
| 421000 | 荆州市                         |    420000 |
| 421100 | 黄冈市                         |    420000 |
| 421200 | 咸宁市                         |    420000 |
| 421300 | 随州市                         |    420000 |
| 422800 | 恩施土家族苗族自治州           |    420000 |
| 429000 | 省直辖行政单位                 |    420000 |
+--------+--------------------------------+-----------+

# 
mysql> select * 
    -> from booktest_areas as province
    -> inner join booktest_areas as city on city.parent_id=province.id
    # 没有下面的where会将区也搜出来
    -> where province.parent_id is null;

# 
mysql> select province.id,province.name,city.name,city.id 
    -> from booktest_areas as province
    -> inner join booktest_areas as city on city.parent_id=province.id
    -> where province.parent_id is null and province.`name`='河北省';
+--------+-----------+--------------+--------+
| id     | name      | name         | id     |
+--------+-----------+--------------+--------+
| 130000 | 河北省    | 石家庄市     | 130100 |
| 130000 | 河北省    | 唐山市       | 130200 |
| 130000 | 河北省    | 秦皇岛市     | 130300 |
| 130000 | 河北省    | 邯郸市       | 130400 |
| 130000 | 河北省    | 邢台市       | 130500 |
| 130000 | 河北省    | 保定市       | 130600 |
| 130000 | 河北省    | 张家口市     | 130700 |
| 130000 | 河北省    | 承德市       | 130800 |
| 130000 | 河北省    | 沧州市       | 130900 |
| 130000 | 河北省    | 廊坊市       | 131000 |
| 130000 | 河北省    | 衡水市       | 131100 |
+--------+-----------+--------------+--------+

# 查询省的名称为“山西省”的所有城市
mysql> select province.*, city.*
    -> from booktest_areas as city
    -> inner join booktest_areas as province on city.parent_id=province.id
    -> where province.`name`='山西省';
+--------+-----------+-----------+--------+-----------+-----------+
| id     | name      | parent_id | id     | name      | parent_id |
+--------+-----------+-----------+--------+-----------+-----------+
| 140000 | 山西省    |      NULL | 140100 | 太原市    |    140000 |
| 140000 | 山西省    |      NULL | 140200 | 大同市    |    140000 |
| 140000 | 山西省    |      NULL | 140300 | 阳泉市    |    140000 |
| 140000 | 山西省    |      NULL | 140400 | 长治市    |    140000 |
| 140000 | 山西省    |      NULL | 140500 | 晋城市    |    140000 |
| 140000 | 山西省    |      NULL | 140600 | 朔州市    |    140000 |
| 140000 | 山西省    |      NULL | 140700 | 晋中市    |    140000 |
| 140000 | 山西省    |      NULL | 140800 | 运城市    |    140000 |
| 140000 | 山西省    |      NULL | 140900 | 忻州市    |    140000 |
| 140000 | 山西省    |      NULL | 141000 | 临汾市    |    140000 |
| 140000 | 山西省    |      NULL | 141100 | 吕梁市    |    140000 |
+--------+-----------+-----------+--------+-----------+-----------+

# 查询咸宁市
mysql> select dis1.*,dis2.*
    -> from booktest_areas as dis1
    -> left join booktest_areas as dis2 on dis2.parent_id=dis1.id
    -> inner join booktest_areas as city on city.id=dis1.parent_id
    -> where city.`name`='咸宁市';
+--------+-----------+-----------+--------+-----------+-----------+
| id     | name      | parent_id | id     | name      | parent_id |
+--------+-----------+-----------+--------+-----------+-----------+
| 421201 | 市辖区    |    421200 | 421202 | 咸安区    |    421201 |
| 421221 | 嘉鱼县    |    421200 |   NULL | NULL      |      NULL |
| 421222 | 通城县    |    421200 |   NULL | NULL      |      NULL |
| 421223 | 崇阳县    |    421200 |   NULL | NULL      |      NULL |
| 421224 | 通山县    |    421200 |   NULL | NULL      |      NULL |
| 421281 | 赤壁市    |    421200 |   NULL | NULL      |      NULL |
+--------+-----------+-----------+--------+-----------+-----------+
 
```

## view

对于复杂的查询，在多次使用后，维护是一件非常麻烦的事情;

视图本质就是对查询的一个封装, 代表一个复杂的select语句;简化调用

```bash
# 命名的时候要用到 v_ 开头来区分table
mysql> create view v_stu_score as 
    -> select students.*,scores.score from scores
    -> inner join students on scores.stu_id=students.id;

# show tables居然可以看到view
mysql> show tables;
+-----------------------+
| Tables_in_myAnotherDB |
+-----------------------+
| booktest_areas        |
| scores                |
| v_stu_score           |
| students              |
| subjects              |
+-----------------------+

mysql> select * from v_stu_score;
+----+--------+------------+--------+----------+-------+
| id | name   | birthday   | gender | isDelete | score |
+----+--------+------------+--------+----------+-------+
|  1 | 郭靖   | 1990-02-12 |       |          | 100.0 |
|  1 | 郭靖   | 1990-02-12 |       |          |  99.0 |
|  1 | 郭靖   | 1990-02-12 |       |          |  98.0 |
|  3 | 曹操   | 1999-01-01 |       |          |  68.6 |
|  3 | 曹操   | 1999-01-01 |       |          |  89.9 |
+----+--------+------------+--------+----------+-------+

# 修改view
mysql> alter view v_stu_score as 
    -> select students.*,scores.score from scores
    -> inner join students on scores.stu_id=students.id
    -> where students.gender=1;
```

- 关系数据库：一个table中的每一record的结构都是一样的;
- noSQL: 每一个record的结构可以不一样;

## Transaction

事务(Transaction)主要用于处理操作量大，复杂度高的数据.
> Innodb, bdb数据库引擎的数据库或表才支持事务  
> 保证成批的SQL语句要么全部执行，要么全部不执行

当一个业务逻辑需要多个sql完成时(insert, update, delete,即那些会影响数据的操作)，如果其中某条sql语句出错，则希望整个操作都退回; 目的是保证一次操作(一个业务逻辑，即多个操作)有效; 

数据库引擎:
- ISAM: 检索很快，写入很慢，因为写入的时候整个table加锁; 不支持事务;
- MYISAM: 不支持事务;
- InnoDB: 读较慢，写较快，写入的时候加锁一个record; 可以多个人同时处理一个table

使用事务的情况: 当数据被更改的时候(insert, update, delete)

事务发生的过程:

- `begin`: 读取record到内存的临时表，进行数据更改(insert, update, delete), 然后从`commit`和`rollback`挑一个运行;
- `commit`: 将内存中的临时表更新到record中;
- `rollback`: 毁掉内存中的临时表(相当于放弃begin;之后的所有更改操作)

```bash
mysql> begin;
mysql> update students set name='小郭' where id=1;
mysql> select * from students;#这里的其实只是内存的临时表，如果在其他client也运行这条命令，还是原来的数据;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 小郭      | 1990-02-12 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  6 | 黄蓉      | 1993-03-29 |        |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
| 10 | 周瑜      | 2000-01-22 |       |          |
| 11 | 诸葛瑾    | 1996-02-23 |       |         |
+----+-----------+------------+--------+----------+

mysql> rollback;
mysql> select * from students;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 郭靖      | 1990-02-12 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  6 | 黄蓉      | 1993-03-29 |        |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
| 10 | 周瑜      | 2000-01-22 |       |          |
| 11 | 诸葛瑾    | 1996-02-23 |       |         |
+----+-----------+------------+--------+----------+

# commit用法
mysql> begin;
mysql> update students set name='小郭' where id=1;
mysql> commit;# 将内存的操作应用到物理上
mysql> select * from students;
+----+-----------+------------+--------+----------+
| id | name      | birthday   | gender | isDelete |
+----+-----------+------------+--------+----------+
|  1 | 小郭      | 1990-02-12 |       |          |
|  3 | 曹操      | 1999-01-01 |       |          |
|  4 | 孔明      | 2001-01-01 |       |          |
|  5 | 司马懿    | 2018-08-09 |       |          |
|  6 | 黄蓉      | 1993-03-29 |        |          |
|  7 | 孙权      | 1996-01-21 |       |          |
|  8 | 小龙女    | 1996-01-22 |        |          |
| 10 | 周瑜      | 2000-01-22 |       |          |
| 11 | 诸葛瑾    | 1996-02-23 |       |         |
+----+-----------+------------+--------+----------+
```

实际事务的使用:

在python(cpp, java)操作数据库的时候，操作失败，会自动rollback, 默认会启用**事务**功能所以一般不用管;

## Index

一般的应用系统，读写比例在`10:1`左右; inser, update很少出现性能问题;复杂查询容易出现问题, 所以优化查询时重中之重;

当数据库数据量很大, 查询很慢,索引能够提高数据访问性能;

primary key和唯一索引都是索引，可以提高速度;
> 索引是对数据库表中一列或多列的值进行排序的一种结构; 数据库里面的索引实际上用的是B+树来实现，能较快的查找数据;

选择列的数据类型:
- 越小的数据类型在磁盘、内存、CPU缓存中更少的空间，处理更快
- 简单数据类型更好，`bit>int>char`
- 避免`null`, 尽量`not null`;MySQL中含有空值的列很难进行查询优化(因为它们使得索引、索引统计、比较运算更加复杂. 一般用0, 一个特殊值、`""`来代替空值)

索引: 针对现有的数据，整理出一块东西用于加速查询，这块东西就是索引;
- 用于快速找到数据
- 数据默认按照某种索引来存
- 索引的创建会增加物理的开销

比如字典就有拼音索引和部首索引，默认是按照拼音索引来存(数据库中是按照primary key来存的)，物理开销就是开头的几页纸；

数据库刚刚开始的时候，没有大量的数据基础，不用刻意做索引；等积累了一定的数据量，访问变慢(>100ms)就要考虑优化(分析业务逻辑的sql语句，优化where后面，给查询频繁的字段加上索引键，也可以给每个字段建立索引，也可以给多个字段建立一个索引)

- 单列索引: 一个索引只包含单个列
- 组合索引: 一个索引包含多个列, 这多个列能够唯一地确定某条记录

```sql
-- 给gender, birthday isDelete建一个索引
-- where gender=0 and birthday>'1990-1-1' and isDelete=0
-- 索引碰到范围，索引失效，后面的索引也是白建的
-- 优化成，将范围放在最后
-- 如果最后一个是or, 索引的作用也失效了, 因为最后一个birthday的时间太长，能不用or尽量不用
where gender=0 and isDelete=0 and birthday>'1990-1-1'
```

mongoDB, Redis是内存级的数据库，在内存中检索，很快；

`where`频繁用到哪些字段就在那些字段用索引；

```bash
# 查看索引
mysql> show index from students;

# 创建索引，字符串才会写length, 其他类型一般不写; length小于字符串长度
mysql> create index index_name on table_name(field1_name(length)，field1_name(length));

# 删除索引
mysql> drop index [index_name] on table_name;
```

索引的缺点:

- 虽然提高查询速度，同时降低更新表的速度(update, insert, delete的时候，除了更新表，还要保持索引文件)
- 所以占用物理空间

example:

```bash
# 开启运行时间检测
mysql> set profiling=1;

# 普通查询
mysql> select * from booktest_areas;
mysql> select * from booktest_areas where name='湖北省';

mysql> show profiles;
+----------+------------+-----------------------------------------------------+
| Query_ID | Duration   | Query                                               |
+----------+------------+-----------------------------------------------------+
|        1 | 0.00555300 | select * from booktest_areas                        |
|        2 | 0.00149850 | select * from booktest_areas where name='湖北省'    |
+----------+------------+-----------------------------------------------------+

# 新增索引之后查询
mysql> desc booktest_areas;
+-----------+-------------+------+-----+---------+----------------+
| Field     | Type        | Null | Key | Default | Extra          |
+-----------+-------------+------+-----+---------+----------------+
| id        | int(11)     | NO   | PRI | NULL    | auto_increment |
| name      | varchar(20) | YES  |     | NULL    |                |
| parent_id | int(11)     | YES  | MUL | NULL    |                |
+-----------+-------------+------+-----+---------+----------------+

mysql> create index name_index on booktest_areas(name(20));
Query OK, 0 rows affected (0.04 sec)

mysql> select * from booktest_areas;
mysql> select * from booktest_areas where name='湖北省';

mysql> show profiles;
+----------+------------+-----------------------------------------------------+
| Query_ID | Duration   | Query                                               |
+----------+------------+-----------------------------------------------------+
|        1 | 0.00555300 | select * from booktest_areas                        |
|        2 | 0.00149850 | select * from booktest_areas where name='湖北省'    |
|        3 | 0.00034725 | show index from booktest_areas                      |
|        4 | 0.00056850 | desc booktest_areas                                 |
|        5 | 0.02790875 | create index name_index on booktest_areas(name(20)) |
|        6 | 0.00051600 | select * from booktest_areas where name='湖北省'    |
|        7 | 0.00511375 | select * from booktest_areas                        |
+----------+------------+-----------------------------------------------------+

# 有关name的速度提高了，其他的速度没有提高；数据量越大，效果越明显

# 关闭检测
mysql> set profiling=0;
```

## final example

```sql
-- 设计班级表，与学生表关联，并进行查询
create table class(
    id int auto_increment primary key not null,
    name varchar(10)
);

create table students(
    id int auto_increment primary key not null,
    name varchar(10),
    class_id int,
    foreign key(class_id) references class(id)
);

create view v_stu_class as
select students.id, students.name, class.name as class_name
from students
inner join class on class.id=students.class_id;
```

```sql
-- 设计分类表(类似京东商品分类)，自关联，并进行查询
-- 分类表是自关联表
create table T_type(
    id int auto_increment primary key not null,
    name varchar(10),
    pid int,
    foreign key(pid) references T_type(id)
);

create view V_type as
select son.* from T_type as father
innder join T_type as son on father.id=sun.id
where father.name='家电';
```

## 常用内置函数

### 字符串函数

```bash
mysql> select ascii('a');
+------------+
| ascii('a') |
+------------+
|         97 |
+------------+

mysql> select char(65);
+----------+
| char(65) |
+----------+
| A        |
+----------+

mysql> select length('hello');
+-----------------+
| length('hello') |
+-----------------+
|               5 |
+-----------------+

mysql> select concat(12,34,'ab');
+--------------------+
| concat(12,34,'ab') |
+--------------------+
| 1234ab             |
+--------------------+

mysql> select left('hello',2);
+-----------------+
| left('hello',2) |
+-----------------+
| he              |
+-----------------+

mysql> select right('hello',2);
+------------------+
| right('hello',2) |
+------------------+
| lo               |
+------------------+

# select substring(str, begin, length)
mysql> select substring('helloworld', 2, 4);
+-------------------------------+
| substring('helloworld', 2, 4) |
+-------------------------------+
| ello                          |
+-------------------------------+

#...见gitbook
```

## MySQL with Python

安装[package](https://stackoverflow.com/questions/4960048/how-can-i-connect-to-mysql-in-python-3-on-windows?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

`pip install mysqlclient`

默认情况下是开启了事务的；

可以用python进行`create table`, 但是一般不这么做，都是先建立好table然后python读写数据；

```python
# insert
import MySQLdb
import random

conn=MySQLdb.connect(host='192.168.128.133',port=3306,db='python3',user='root',passwd='13810455459',charset='utf8')

def interact_mysql(sql_cmd):
    try:
        cusor1=conn.cursor()
        # 执行成功返回1
        content=cusor1.execute(sql_cmd)
        print(content)
        conn.commit()
        cusor1.close()
    except Exception as e:
        print(f'errro:{e}')

def main():
    for i in range(10):
        interact_mysql(f"insert into students(name,gender,birthday) values('person-{i+1}',{random.randint(0,1)},'1990-1-{i+1}')")
    conn.close()

if __name__ == '__main__':
    main()
```

pycharm中`ctrl+click`查看函数；

```python
# update
def main():
    interact_mysql(f"update students set name='boy' where gender=1")
    conn.close()
```

```python
# delete
def main():
    interact_mysql('delete from students where gender=1')
    conn.close()
```

sql语句参数化: 为了数据库的安全性，防止sql注入，比如`select * from students where name=''`，别人填上`a' or 1=1 or '`

```python
import MySQLdb

conn=MySQLdb.connect(host='192.168.128.133',port=3306,db='python3',user='root',passwd='13810455459',charset='utf8')

def interact_mysql(sql_cmd, params):
    try:
        cusor1=conn.cursor()
        cusor1.execute(sql_cmd, params)

        # # 显示一行查询结果
        # row=cusor1.fetchone()
        # print(row)
        # 显示所有查询结果
        rows=cursor1.fetchall()
        print(rows)

        conn.commit()
        cusor1.close()
    except Exception as e:
        print(f'errro:{e}')

def main():
    query_name=input('Enter your query name and gender: ')
    args=query_name.split(' ')
    print(args)
    # 无论什么数据类型都是用%s
    interact_mysql('select * from students where name=%s and gender=%s',args)
    conn.close()

if __name__ == '__main__':
    main()
```

```bash
#output
Enter your query name and gender: person-5 0
['person-5', '0']
(47, 'person-5', b'\x00', datetime.date(1990, 1, 5), b'\x00')
```

### 封装

```python
import MySQLdb
import random


class MysqlHelper(object):
    def __init__(self, host, port, db, user, passwd, charset='utf8'):
        self.host = host
        self.port = port
        self.db = db
        self.user = user
        self.passwd = passwd
        self.charset = charset
        # 不在这里面connect(),尽量晚打开，早关闭

    def open(self):
        self.conn = MySQLdb.connect(host=self.host, port=self.port, db=self.db,
                                    user=self.user, passwd=self.passwd, charset=self.charset)
        self.cursor = self.conn.cursor()

    def close(self):
        self.cursor.close()
        self.conn.close()

    # crud: r,retrive
    # create, update, delete
    def cud(self, sql, params):
        try:
            self.open()
            self.cursor.execute(sql, params)
            self.conn.commit()
        except Exception as e:
            print(f'error:{e}')
        finally:
            self.close()

    def retrive(self, sql, params=[]):
        try:
            self.open()
            self.cursor.execute(sql, params)
            res = self.cursor.fetchall()

            self.conn.commit()
            return res
        except Exception as e:
            print(f'error: {e}')
        finally:
            self.close()


def main():
    helper = MysqlHelper('192.168.128.133', 3306, 'python3',
                         'root', '13810455459', 'utf8')
    args = ['person-5', '0']
    print(helper.retrive('select * from students where name=%s and gender=%s', args))

    another_args=['grey',1,'1991-5-6']
    helper.cud("insert into students(name,gender,birthday) values(%s,%s,%s)",another_args)


if __name__ == '__main__':
    main()
```

```bash
#output
((47, 'person-5', b'\x00', datetime.date(1990, 1, 5), b'\x00'),)
```

### python交互实例-用户登录

用户登录其实是一个查询操作，注册才是insert操作；

![](res/login01.png)

```bash
# 创建数据库
mysql> desc userInfos;
+----------+------------------+------+-----+---------+----------------+
| Field    | Type             | Null | Key | Default | Extra          |
+----------+------------------+------+-----+---------+----------------+
| id       | int(11) unsigned | NO   | PRI | NULL    | auto_increment |
| uname    | varchar(20)      | NO   |     | NULL    |                |
| upwd     | char(40)         | NO   |     | NULL    |                |
| isDelete | bit(1)           | NO   |     | b'0'    |                |
+----------+------------------+------+-----+---------+----------------+
# 已知了sha1的加密是40位所以用char(40),而不是varchar(40)
```

采用`sha1`加密(40位)，比`md5`安全(32位)

```python
# insert
def main():
    uname, pwd_string = 'grey', '123456'.encode('utf-8')
    upwd = hashlib.sha1(pwd_string).hexdigest()
    # print(upwd, len(upwd))

    helper = MysqlHelper('192.168.128.133', 3306, 'myAnotherDB',
                         'root', '13810455459', 'utf8')
    args = [uname, upwd]
    helper.cud('insert into userInfos(uname,upwd) values(%s,%s)', args)
```

```python
# query
def main():
    # enter uname, upwd
    uname, pwd_string = 'tim', '1234567'.encode('utf-8')
    upwd = hashlib.sha1(pwd_string).hexdigest()
    helper = MysqlHelper('192.168.128.133', 3306, 'myAnotherDB',
                         'root', '13810455459', 'utf8')
    args = [uname, ]

    res=helper.retrive('select upwd from userInfos where uname=%s', args)
    if len(res)==0:
        print('user name error!')
    elif res[0][0]==upwd:
        print('login success!')
    else:
        print('password error')
```

注册功能也是好做的，主要是要求用户名不重复；


## sqlalchemy

没有ORM使用原生的sql语句缺点: 
- 需要拼接sql字符串; 
- 如果数据库结构改变，源代码就要跟着修改，扩展不方便;
- 不是专业的DBA, 拼接的原生sql语句可能不高效;

sqlalchemy: SQLAlchemy是Python编程语言下的一款**ORM**框架，该框架建立在数据库API之上，使用关系对象映射进行数据库操作.
> 将对象转换成SQL，然后使用数据API执行SQL并获取执行结果  
> 不需要写sql语句，而是使用面向对象中的`.`来操作对象;

[sqlalchemy+mysqlclient](https://docs.sqlalchemy.org/en/latest/dialects/mysql.html)
   ```python
   engine = create_engine("mysql+mysqldb://root:123456@localhost/world", echo=True)

   # 支持中文,首先要求database china支持utf8
   engine = sqlalchemy.create_engine(
    "mysql+mysqldb://root:123456@localhost/china?charset=utf8mb4&binary_prefix=true", echo=True)
   ```

example: sqlalchemy create table & insert record

```python
import sqlalchemy
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext import declarative

Base = declarative.declarative_base()

class User(Base):
    __tablename__ = 'table_name'
    id = Column(Integer, primary_key=True)
    name = Column(String(32))
    password = Column(String(64))

    def __repr__(self):
        return f"<User(name='{self.name}', password='{self.password}')>"


engine = sqlalchemy.create_engine(
    "mysql+mysqldb://grey:xxxxxx@localhost/world", echo=True)

# create table
Base.metadata.create_all(engine)

# insert
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

for i in range(3):
    u = User(name=f'grey-{i+1}', password=f'pwd-{i**2}')
    print(u)
    s.add(u)
s.commit()
```

example: sqlalchemy query

```python
# without filter
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()
# query data
u = s.query(User).all()  # object list
print(u) # [<User(name='grey-1', password='pwd-0')>, <User(name='grey-2', password='pwd-1')>, <User(name='grey-3', password='pwd-4')>]

u = s.query(User.name, User.password).all()
print(u) # [('grey-1', 'pwd-0'), ('grey-2', 'pwd-1'), ('grey-3', 'pwd-4')]
```

```python
# with filter
u1 = s.query(User).filter(User.id > 11).all()
print(u1) # [<User(name='grey-3', password='pwd-4')>]
u2 = s.query(User).filter(User.name == 'grey-2').all()
print(u2) # [<User(name='grey-2', password='pwd-1')>]
u3 = s.query(User).filter(User.id > 11).filter(User.id < 13).all()
print(u3) # [<User(name='grey-3', password='pwd-4')>]
```

example: sqlalchemy modify data

```python
# modify data
u = s.query(User).filter(User.id > 11).filter(User.id < 13).first()
print(u)
u.name='james'
u.password='hahaha'
s.commit()
```

example: sqlalchemy groupby

```python
N = s.query(User).filter(User.name.like('Gre%')).count()
print(N) # 2

# navicat add some data, then
from sqlalchemy import func
u=s.query(User.name, func.count(User.name)).group_by(User.name).all()
print(u) # [('grey-1', 1), ('grey-2', 1), ('james', 2), ('moriaty', 1)]
```

example: multiple tables

```python
# Create 2 tables
import sqlalchemy
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.ext import declarative

Base = declarative.declarative_base()

class Student(Base):
    __tablename__ = 'students'
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(32))

    def __repr__(self):
        return f'<id={self.id},name={self.name}>'


class Score(Base):
    __tablename__ = 'scores'
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(32), nullable=False)
    value = Column(Integer)
    stu_id = Column(Integer, ForeignKey('students.id'))

    def __repr__(self):
        return f'<subject={self.name}, score={self.value}>'


engine = sqlalchemy.create_engine(
    "mysql+mysqldb://grey:xxxxxx@localhost/world", echo=True)

# create 2 tables
Base.metadata.create_all(engine)
```

```python
# join tables
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()
# filter method: 可以不用ForeignKey
r1 = s.query(Student.name, Score.name, Score.value).filter(Student.id==Score.stu_id).all()
print(r1)
# join method
r2 = s.query(Student.name, Score.name, Score.value).join(Score).all()
print(r2)
```

example: back reference(反查)

```python
import sqlalchemy
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.ext import declarative

Base = declarative.declarative_base()


class Student(Base):
    __tablename__ = 'students'
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(32))

    def __repr__(self):
        return f'<id={self.id},name={self.name}>'


class Score(Base):
    __tablename__ = 'scores'
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(32), nullable=False)
    value = Column(Integer)
    stu_id = Column(Integer, ForeignKey('students.id'))

    # scores表中可以通过stu字段到students, 也可以在students表中通过my_score到scores表中
    # 这一条没有在数据库中创建,只是告诉两者的关系
    stu = sqlalchemy.orm.relationship('Student', backref='my_score')

    def __repr__(self):
        return f'<subject={self.name}, score={self.value}, stu={self.stu.name}>'


engine = sqlalchemy.create_engine(
    "mysql+mysqldb://grey:xxxxxx@localhost/world", echo=True)

Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

u1 = s.query(Student).filter(Student.name == 'Grey').first()
print(u1.my_score) # [<subject=Math, score=100, stu=Grey>, <subject=English, score=86, stu=Grey>, <subject=Physics, score=56, stu=Grey>]
```

example: multiple foreign keys

```python
import sqlalchemy
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.ext import declarative

Base = declarative.declarative_base()


class Address(Base):
    __tablename__ = 'addresses'
    id = Column(Integer, primary_key=True)
    city = Column(String(32))


class Customer(Base):
    __tablename__ = 'customers'
    id = Column(Integer, primary_key=True)
    name = Column(String(32))

    billing_address_id = Column(Integer, ForeignKey("addresses.id"))
    shipping_address_id = Column(Integer, ForeignKey("addresses.id"))

    # 当存在多个外键时，foreign_keys参数是为了插入数据找到对应关系，必须要有
    billing_address = sqlalchemy.orm.relationship("Address", foreign_keys=[billing_address_id])
    shipping_address = sqlalchemy.orm.relationship("Address", foreign_keys=[shipping_address_id])


engine = sqlalchemy.create_engine(
    "mysql+mysqldb://grey:xxxxxx@localhost/world", echo=True)
# Base.metadata.create_all(engine)

Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

# add data
addr1=Address(city='Hongkong')
addr2=Address(city='New York')
addr3=Address(city='Tokyo')
s.add_all([addr1, addr2, addr3])

c1=Customer(name='xiaoming', billing_address_id=1, shipping_address_id=3)
c2=Customer(name='lihua', billing_address_id=2, shipping_address_id=1)
c3=Customer(name='hanmei', billing_address_id=2, shipping_address_id=3)
s.add_all([c1, c2, c3])

s.commit()
```

example: [多对多关系](http://www.cnblogs.com/alex3714/articles/5978329.html)

```python
import sqlalchemy
from sqlalchemy import Table, Column, Integer, String, ForeignKey
from sqlalchemy.ext.declarative import declarative_base


Base = declarative_base()


# 通过这张中间表将authors和books的多对多关系变成两个一对多
# 用户不用手动去管理插数据，ORM自动维护；所以用这种简单的方式来创建表；mysql数据库是真实存在的
book_m2m_author = Table('book_m2m_author', Base.metadata,
                        Column('book_id', Integer, ForeignKey('books.id')),
                        Column('author_id', Integer, ForeignKey('authors.id')),
                        )


class Author(Base):
    __tablename__ = 'authors'
    id = Column(Integer, primary_key=True)
    name = Column(String(32))

    def __repr__(self):
        return self.name


class Book(Base):
    __tablename__ = 'books'
    id = Column(Integer, primary_key=True)
    name = Column(String(64))
    pub_date = Column(String(32))

    # secondary表示通过第三方来关联Author
    aus = sqlalchemy.orm.relationship(
        'Author', secondary=book_m2m_author, backref='books')

    def __repr__(self):
        return f'<book={self.name} in {self.pub_date}, authors={self.aus}>'


engine = sqlalchemy.create_engine(
    "mysql+mysqldb://grey:xxxxxx@localhost/world", echo=True)
Base.metadata.create_all(engine)

Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

a1 = Author(name='Moris')
a2 = Author(name='Jack')
a3 = Author(name='Tom')
s.add_all([a1, a2, a3])

b1 = Book(name='Red', pub_date='2017', aus=[a1, a3])
b2 = Book(name='Green', pub_date='2016', aus=[a2, a3])
b3 = Book(name='Blue', pub_date='2018', aus=[a1, a3])
s.add_all([b1, b2, b3])

s.commit()
```

```python
# query
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

b1 = s.query(Book).filter(Book.name == 'Red').first()
print(b1) # <book=Red in 2017, authors=[Moris, Tom]>
a1 = s.query(Author).filter(Author.name == 'Moris').first()
print(a1, a1.books) # [<book=Red in 2017, authors=[Moris, Tom]>, <book=Blue in 2018, authors=[Moris, Tom]>]
```

```python
# remove
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

b1 = s.query(Book).filter(Book.name == 'Red').first()
a1 = s.query(Author).filter(Author.name == 'Moris').first()
# 将a1作者从b1书中删除
b1.aus.remove(a1)
s.commit()
```

```python
# delete所有与书有关的关联
Session_class = sqlalchemy.orm.sessionmaker(bind=engine)
s = Session_class()

b1 = s.query(Book).filter(Book.name == 'Red').first()
s.delete(b1)
s.commit()
```

[Homework](http://www.cnblogs.com/alex3714/articles/5978329.html)