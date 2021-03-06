## 数据类型

只罗列一些比较常见的

#### 整型

* ```tinynt```

* ```mediumint```

* ```smalliny```

* ```shortint```

* ```int```

* ```integer```

#### 精度类型，可以保留小数

* ```dacimal(m, n)```

#### 日期类型

* ```date```

#### 日期时间类型

* ```datetime```

#### 固定字符串

* ```char(n)```

#### 变长字符串

* ```varchar```

#### 变长的文本类型

* ```text```

#### 大的文本类型，0 ~ 4G

* ```longtext```

#### 枚举类型（enum("男", "女", "保密")）（用的较少）

* ```enum```

----

----


## 数据类型整理如下

* ```id``` 类型一般使用 ```int```，如果知道以后 ```id``` 会特别大，就用 ```bigint```（可以使用 ```auto_increment``` 来实现自增加）

* 用户姓名，用户的 ```id``` 一类（例如 ```userId```），用户地址等一类用 ```varchar```（```varchar(64)```）或者 ```char```，```text``` 字段一般不建议使用，使用时建议一张表单独用来存这个字段 

* 是 和 否 可以用 ```tinyint```，```smallint``` 也可以，类似存储性别（男，女）一类的，也可以使用这个

* 日期类型就 ```date``` 或者 ```datetime```，也可以存储时间的 ```unix``` 时间戳格式（时间戳只有 ```10``` 位用 ```int``` 就可以了）

有一个问题，关于表类型，因为可以随时修改，为什么不在需要相应类型的时候直接调整

这是因为字段类型是可以修改，在表数据量小的情况下，改数据类型很方便，但是如果数据量大了，就涉及到锁表了，这就是 ```dba``` 对数据库维护的问题了

说白了，改一个字段的字段类型，有可能是先复制一张这个表作为临时表，然后把临时表里这个字段的字段类型改掉，还要先检查这个字段对应的值能不能被修改成目标类型，比如原先这个字段是 ```varchar``` 表里数据都存的 ```abc```，你要改成 ```int``` 就改不了

数据量越大 索引越多的表 改字段就要考虑性能了


----

----


#### 查看建立数据库命令的代码

```js
show create database yy;

// ===> 会返回

create database `yy` /*!40100 default character set utf8 */

// <=== 实际上等价于 ===> (/**/ 的意思为在 mysql 4.01 以下不支持多国语言)

create database `yy` default character set utf8;
```

----

----


## 数据库的增删改查

#### 创建数据库（增）

```js
// 常规方式
create databse yy;

// 使用 `` 避免库名（兼容性）
create database `database`;

// 判断数据库 yy 是否存在，如果不存在才创建此数据库，并且设定新数据库的字符集编码为 utf8 格式
create database if not exists yy default character set utf8;
```



#### 删除数据库（删）

```js
drop database yy;

drop database if exists yy;

drop database if exists `yy`;
```



#### 修改数据库（改）

```js
// 设定数据库的默认字符集
alter database yy default character set utf8;
```

需要注意的是：若是要修改数据库的名称的话，请先备份数据（这个操作很危险）

大体思路是借助 ```rename``` 这个命令

```js
// 使用
rename olddb.tables to newdb.tables
```

但是这个命令碰到视图会出错，也不能把旧库中的函数，存储过程，事件移过来，对于这一些，我们可以从旧库中导出这些定义，再导入到新库中




#### 查看所有数据库（查）

```js
show databases;
```


----

----




## 表的增删改查

#### 建立表（增）

```js
create table if not exists y1(
    yid int unsigned not null auto_increment,
    yname varchar(30) not null,
    primary key(yid)
)engine = myisam auto_increment = 100 default charset utf8;

```

```if not exists```

* 如果不存在再去创建

```unsigned```

* ```int``` 分为无符号 ```unsigned``` 和有符号 ```signed``` 两种类型，默认为 ```signed```

* 二者的区别就是无符号类型能保存 ```2``` 倍于有符号类型的数据

```not null```

* 表示不为空

```auto_increment```

* 自增加，比如 ```id``` 添加数据后自动变为 ```1234..```


```engine```

* ```engine``` 表示存储引擎，一般分为 ```myisam```，```innodb``` 等，```MySQL 5.5``` 起，默认为 ```innodb```，但是我们大部分时候都采用 ```myisam```（若是不涉及事务的话，选用这个速度较快），```innodb``` 一般用于事务（比如票务查询，银行业务转账）

* ```engine``` 中的 ```auto_increment``` 表示编号字段默认从 ```100``` 开始（上例中），即 ```101, 102, 103...```

#### 删除表（删）

同库的操作类似

```js
drop table y1;

drop table if exists y1;

// 删除多个表
drop table y1, y2, y3;
```



#### 修改表的类型及字段名（改）


修改表名（一般用的较少）

```js
rename table y1 to y2;

// <===等价于===>

alter table y2 rename y1;
```


修改表结构

```js
// 先查看一下表结构
desc y1;

// ===>

+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| yid   | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| yname | varchar(30)      | NO   |     | NULL    |                |
+-------+------------------+------+-----+---------+----------------+

// 修改字段名并重新指定数据类型
alter table y1 change yid userid int;

alter table y1 change yname username varchar(30) not null;


// 修改字段类型
alter table 表名 monify 字段名 字段类型;

// ===>

+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| userid   | int(11)     | NO   | PRI | 0       |       |
| username | varchar(30) | NO   |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
```

新增一个字段

```js
// 新增的时候可以指定插入的位置，只需要使用 after 关键字
alter table y1 add usertel varchar(20);

alter table y1 add useraddress varchar(50) after username;

// ===>

+-------------+-------------+------+-----+---------+-------+
| Field       | Type        | Null | Key | Default | Extra |
+-------------+-------------+------+-----+---------+-------+
| userid      | int(11)     | NO   | PRI | 0       |       |
| username    | varchar(30) | NO   |     | NULL    |       |
| useraddress | varchar(50) | YES  |     | NULL    |       |
| usertel     | varchar(20) | YES  |     | NULL    |       |
+-------------+-------------+------+-----+---------+-------+
```

删除一个字段

```js
alter table y1 drop column usertel;

// ===>

+-------------+-------------+------+-----+---------+-------+
| Field       | Type        | Null | Key | Default | Extra |
+-------------+-------------+------+-----+---------+-------+
| userid      | int(11)     | NO   | PRI | 0       |       |
| username    | varchar(30) | NO   |     | NULL    |       |
| useraddress | varchar(50) | YES  |     | NULL    |       |
+-------------+-------------+------+-----+---------+-------+
```



#### 查询表（查）

```js
// 查看表
show tables;

// ===>
+--------------+
| Tables_in_yy |
+--------------+
| stu          |
+--------------+

// 查看表结构
desc stu;

// ===>

+----------+---------------------+------+-----+---------+----------------+
| Field    | Type                | Null | Key | Default | Extra          |
+----------+---------------------+------+-----+---------+----------------+
| id       | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| name     | varchar(30)         | NO   |     | NULL    |                |
| age      | tinyint(3) unsigned | NO   |     | NULL    |                |
| birthday | datetime            | YES  |     | NULL    |                |
| address  | varchar(50)         | YES  |     | NULL    |                |
| score    | decimal(4,1)        | YES  |     | NULL    |                |
+----------+---------------------+------+-----+---------+----------------+

```




----

----






## 表结构的增删改查

#### 数据的插入（增）

```js
// 新建立一个学生表
create table stu(
    id int unsigned not null auto_increment,
    name varchar(30) not null,
    age tinyint unsigned not null,
    birthday datetime,
    address varchar(50),
    score decimal(4, 1),
    primary key(id)
)engine=myisam auto_increment = 201201 default charset=utf8;

// ===> 

+----------+---------------------+------+-----+---------+----------------+
| Field    | Type                | Null | Key | Default | Extra          |
+----------+---------------------+------+-----+---------+----------------+
| id       | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| name     | varchar(30)         | NO   |     | NULL    |                |
| age      | tinyint(3) unsigned | NO   |     | NULL    |                |
| birthday | datetime            | YES  |     | NULL    |                |
| address  | varchar(50)         | YES  |     | NULL    |                |
| score    | decimal(4,1)        | YES  |     | NULL    |                |
+----------+---------------------+------+-----+---------+----------------+
```

关于 ```birthday```

一般不直接存储年龄，而是存储出生日期


```insert``` （插入数据）

```js
insert into stu values(null, "张三", "18", "2005-2-4", "bj", "65");

// into 可以省略（仅限 mysql 中，oracle 中不加则报错，建议全都加上）
insert stu values(null, "李四", "19", "2006-6-6", "sh", "68.5");

// 也可以一次插入多条数据
insert stu values(null, "王五", "20", "20106-7-8", "sz", "90"), (null, "赵六", "19", "1996-5-6", "gz", "80.5");

// 也可以只插入部分数据
insert stu(name, age, score) values("孙七", "18", "85"), ("钱八", "22", "88");

// replace 插入
replace into stu(name, age, score) values("梁九", "15", "76"), ("刘十", "26", 95);

// replace 与 insert 的区别
// replace 可以插入也可以替换，在主键存在的情况下，replace 会直接替换数据
replace into stu values(201206, "真钱八", "28", "2018-08-18", "ss", "98");

// ===>

+--------+--------+-----+---------------------+---------+-------+
| id     | name   | age | birthday            | address | score |
+--------+--------+-----+---------------------+---------+-------+
| 201201 | 张三   |  18 | 2005-02-04 00:00:00 | bj      |  60.0 |
| 201202 | 李四   |  19 | 2006-06-06 00:00:00 | sh      |  60.5 |
| 201203 | 王五   |  20 | 0000-00-00 00:00:00 | sz      |  60.5 |
| 201204 | 赵六   |  19 | 1996-05-06 00:00:00 | gz      |  60.5 |
| 201205 | 孙七   |  18 | NULL                | NULL    |  NULL |
| 201206 | 真钱八 |  28 | 2018-08-18 00:00:00 | ss      |  98.0 |
| 201207 | 梁九   |  15 | NULL                | NULL    |  NULL |
| 201208 | 刘十   |  26 | NULL                | NULL    |  NULL |
+--------+--------+-----+---------------------+---------+-------+

// 还可以复制表，即把当前表内容复制一份在插入其中
insert into stu(name, age, birthday, address, score) select name, age, birthday, address, score from stu;

// ===>

+--------+--------+-----+---------------------+---------+-------+
| id     | name   | age | birthday            | address | score |
+--------+--------+-----+---------------------+---------+-------+
| 201201 | 张三   |  18 | 2005-02-04 00:00:00 | bj      |  60.0 |
| 201202 | 李四   |  19 | 2006-06-06 00:00:00 | sh      |  60.5 |
| 201203 | 王五   |  20 | 0000-00-00 00:00:00 | sz      |  60.5 |
| 201204 | 赵六   |  19 | 1996-05-06 00:00:00 | gz      |  60.5 |
| 201205 | 孙七   |  18 | NULL                | NULL    |  NULL |
| 201206 | 真钱八 |  28 | 2018-08-18 00:00:00 | ss      |  98.0 |
| 201207 | 梁九   |  15 | NULL                | NULL    |  NULL |
| 201208 | 刘十   |  26 | NULL                | NULL    |  NULL |
| 201209 | 张三   |  18 | 2005-02-04 00:00:00 | bj      |  60.0 |
| 201210 | 李四   |  19 | 2006-06-06 00:00:00 | sh      |  60.5 |
| 201211 | 王五   |  20 | 0000-00-00 00:00:00 | sz      |  60.5 |
| 201212 | 赵六   |  19 | 1996-05-06 00:00:00 | gz      |  60.5 |
| 201213 | 孙七   |  18 | NULL                | NULL    |  NULL |
| 201214 | 真钱八 |  28 | 2018-08-18 00:00:00 | ss      |  98.0 |
| 201215 | 梁九   |  15 | NULL                | NULL    |  NULL |
| 201216 | 刘十   |  26 | NULL                | NULL    |  NULL |
+--------+--------+-----+---------------------+---------+-------+
```


#### 删除数据（删）

```js
delete from stu where id = 201216;

// ===>

+--------+--------+-----+---------------------+---------+-------+
| id     | name   | age | birthday            | address | score |
+--------+--------+-----+---------------------+---------+-------+
| 201201 | 张三   |  18 | 2005-02-04 00:00:00 | bj      |  60.0 |
| 201202 | 李四   |  19 | 2006-06-06 00:00:00 | sh      |  60.5 |
| 201203 | 王五   |  20 | 2008-08-08 00:00:00 | wh      |  80.0 |
| 201204 | 赵六   |  19 | 1996-05-06 00:00:00 | gz      |  60.5 |
| 201205 | 孙七   |  18 | NULL                | NULL    |  NULL |
| 201206 | 真钱八 |  28 | 2018-08-18 00:00:00 | ss      |  98.0 |
| 201207 | 梁九   |  15 | NULL                | NULL    |  NULL |
| 201208 | 刘十   |  26 | NULL                | NULL    |  NULL |
| 201209 | 张三   |  18 | 2005-02-04 00:00:00 | bj      |  60.0 |
| 201210 | 李四   |  19 | 2006-06-06 00:00:00 | sh      |  60.5 |
| 201211 | 王五   |  20 | 0000-00-00 00:00:00 | sz      |  60.5 |
| 201212 | 赵六   |  19 | 1996-05-06 00:00:00 | gz      |  60.5 |
| 201213 | 孙七   |  18 | NULL                | NULL    |  NULL |
| 201214 | 真钱八 |  28 | 2018-08-18 00:00:00 | ss      |  98.0 |
| 201215 | 梁九   |  15 | NULL                | NULL    |  NULL |
+--------+--------+-----+---------------------+---------+-------+

// 无条件，删除所有记录
delete from stu;

// 还可以直接清空（全部删除）数据表记录，auto_increment 自动编号从 1 重新分配
// 用的较少
truncate table stu;
```



#### update（修改数据）

```js
update stu set score = 80, address = "wh", birthday = "2008-08-08" where id = 201203;

// ===>

+--------+------+-----+---------------------+---------+-------+
| id     | name | age | birthday            | address | score |
+--------+------+-----+---------------------+---------+-------+
| 201203 | 王五 |  20 | 2008-08-08 00:00:00 | wh      |  80.0 |
+--------+------+-----+---------------------+---------+-------+
```


#### 查询数据（查）

```js
// 最基本的查询方式
select * from stu;

// ===>

// 去掉了重复的数据，然后整理更新了一下，用来查询
+--------+------+-----+------------+-------+
| id     | name | age | birthday   | score |
+--------+------+-----+------------+-------+
| 201209 | 张三 |  18 | 2005-02-04 |  65.0 |
| 201210 | 李四 |  19 | 2006-06-06 |  68.5 |
| 201211 | 王五 |  20 | 1995-02-06 |  90.0 |
| 201212 | 赵六 |  19 | 1991-03-16 |  80.5 |
| 201213 | 孙七 |  18 | 1992-05-06 |  85.0 |
| 201214 | 钱八 |  22 | 2002-12-16 |  88.0 |
| 201215 | 孙七 |  18 | 2008-08-08 |  85.0 |
| 201216 | 钱八 |  22 | 1998-08-28 |  88.0 |
| 201217 | 梁九 |  15 | 1988-01-02 |  76.0 |
| 201218 | 刘十 |  26 | 1989-11-22 |  95.0 |
+--------+------+-----+------------+-------+

// 只查询姓名和成绩
select name, score, from stu;

// ===>

+------+-------+
| name | score |
+------+-------+
| 张三 |  65.0 |
| 李四 |  68.5 |
| 王五 |  90.0 |
| 赵六 |  80.5 |
| 孙七 |  85.0 |
| 钱八 |  88.0 |
| 孙七 |  85.0 |
| 钱八 |  88.0 |
| 梁九 |  76.0 |
| 刘十 |  95.0 |
+------+-------+
```

查询的基本条件， ```<```，```>```，```<=```，```=>```，```=```，```!=```，```<>```

```js
// 查询分数小于 80 分的
select name, score from stu where score < 80;

// 等于 90 分的
select name, score from stu where score = 90;

// 省略一些

// 关于 != 和 <> 都表示否定，还有一个 not 也表示否定
select name, score from stu where score != 90;
select name, score from stu where score <> 90;
select name, score from stu where not score = 90;
```

范围查询，```between..and```

```js
// 查询分数在 80 ~ 90 之间，注意：包含 80 和 90
select name, score from stu where score between 80 and 90;

// 查询分数不在 80 ~ 90 之间
select name, score from stu where score not between 80 and 90;

// ===>

+------+-------+
| name | score |
+------+-------+
| 王五 |  90.0 |
| 赵六 |  80.5 |
| 孙七 |  85.0 |
| 钱八 |  88.0 |
| 孙七 |  85.0 |
| 钱八 |  88.0 |
+------+-------+
```

模糊查询 ```like```（```%``` 和 ```_```）

需要注意的是：如果查询语句中没有 ```like```，比如 ```select * from stu where name = "张%";```

查询语句不会报错，但是 ```%``` 不再是任意符号，因为判断的是等号，不再支持通配符

```js
// _ 表示一个任意字符，比如查询名字为两个字的用户
select name from stu where name like "__"

// 查询名字为两个字的而且是姓张的
select name from stu where name like "张_"

// % 表示零个或多个字符，比如查询姓名中含有三这个字的用户
select name from stu where name like "%三%"
```

基本条件 ```in```，后面接一个限定范围

```js
// 查询 id 为 201209, 201210, 201208 的三条数据
select * from stu where id in (201209, 201210, 201208);

// 也可以使用 not in 排除这三条数据
select * from stu where id not in (201209, 201210, 201208);
```

基本条件 ```null```

在查询的时候，比如某个字段的数据为 ```null``` 不能直接使用 ```= null```（还有一个 ```is not null```）

```js
// 不报错，但是查询为空（这个语句尽量不要出现）
select * from stu where score = null;

// 这个时候可以使用 is null
select * from stu where score is null;

// 然后可以批量处理这些数据，比如成绩为 null 的全部设为 0 分
update stu set score = 0 where score is null;


//---- 注意区分 "" 和 null


// 这个表示查询名字为空字符串的
select * from stu where name = "";

// 这个表示查询名子没有填写的
select * from stu where name is null;
```