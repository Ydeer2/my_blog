---
title: "MYSQL的优化"
date: 2026-03-10
categories:
  - 技术
  - 后端
tags:
  - 后端
  - 数据库
  - 笔记
---

# MYSQL的优化

近来无聊，总结了一下mysql优化的相关知识点。

## 慢查询日志

慢查询日志文件可以将查询较慢的DQL语句记录下来，便于我们定位需要优化的select语句。

通过以下命令查看慢查询日志功能是否开启：

```mysql
show variables like 'show_query_log';
```

在MYSQL当中默认是不开启慢查询日志的

需要自己手动配置慢查询日志

![](<https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/82aacf5ac5931a5acdcda7f7dcacc82a%20(1).png>)

在my.ini文件中配置

```ini
#slow_query_log =1 表示开启慢查询日志
slow_query_log=1
#long_query_time = 10 表示当select语句耗时超过10秒之后，会被记录到慢查询日志当中
long_query_time=10
```

这个慢查询日志是存储在Data目录下，默认的名字是：计算机名-slow.log，这个计算机名是你的主机名，在my.ini当中也有配置

配置项是

```ini
slow_query_log_file = 计算机名-slow.log
```

### 演示：

随便执行一个查询语句，在查询的时候调用sleep()函数，伪装慢查询。

```mysql
select * , sleep(10) from emp where ename='smith'
```

查询结果是等于10秒

```tex
+-------+-------+-------+------+------------+--------+------+--------+-----------+
| EMPNO | ENAME | JOB   | MGR  | HIREDATE   | SAL    | COMM | DEPTNO | sleep(10) |
+-------+-------+-------+------+------------+--------+------+--------+-----------+
|  7369 | SMITH | CLERK | 7902 | 1980-12-17 | 800.00 | NULL |     20 |         0 |
+-------+-------+-------+------+------------+--------+------+--------+-----------+
1 row in set (10.00 sec)

```

这是再去查看日志文件，我们用管理员身份打开记事本

可以看到

![](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/19f25c646dd84c5c8cd9afd5abcbecee.png)

这条SQL语句被记录下来了。

## show profiles

通过show profiles；我们可以查看最近执行select语句的耗时情况。

这个profile功能也是默认不开启的，需要我们手动开启，好消息不用去改配置文件，执行下面命令就可以

```mysql
set profiling = 1;
```

通过

```sql
select @@profiling;
```

可以查看profiles功能是否开启

```tex
+-------------+
| @@profiling |
+-------------+
|           1 |
+-------------+
```

这样就是开启了

开启之后执行

```sql
show profiles;
```

可以查看最近执行的select语句的情况

```sql
+----------+------------+--------------------------+
| Query_ID | Duration   | Query                    |
+----------+------------+--------------------------+
|        1 | 0.00030925 | select @@profiling       |
|        2 | 0.00258875 | select count(*) from emp |
|        3 | 0.00295050 | select count(*) from emp |
+----------+------------+--------------------------+
```

也可以查看特定的查询语句的具体耗时情况，执行以下代码

```sql
show profile for query 2; #表示查看Query_ID为2的select语句的耗时情况
```

各个阶段的具体耗时情况就会显示出来。

```sql
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.000111 |
| Executing hook on transaction  | 0.000031 |
| starting                       | 0.000009 |
| checking permissions           | 0.000006 |
| Opening tables                 | 0.000074 |
| init                           | 0.000006 |
| System lock                    | 0.000011 |
| optimizing                     | 0.000008 |
| statistics                     | 0.000025 |
| preparing                      | 0.000020 |
| executing                      | 0.002161 |
| end                            | 0.000012 |
| query end                      | 0.000003 |
| waiting for handler commit     | 0.000014 |
| closing tables                 | 0.000011 |
| freeing items                  | 0.000075 |
| cleaning up                    | 0.000016 |
+--------------------------------+----------+
```

想要了解更多关于show profiles的信息，可以查看官方文档

[MySQL :: MySQL 8.0 Reference Manual :: 15.7.7.30 SHOW PROFILE Statement](https://dev.mysql.com/doc/refman/8.0/en/show-profile.html)

有意思的是官方文档中认为不应该使用这个show profiles，并表示这个命令可以在未来版本会移出，推荐我们使用性能模式的查询分析，两种查询分析的结果是基本一致的，但过程不一样，对这种性能模式的查询分析感兴趣的可以查看

[MySQL :: MySQL 8.0 Reference Manual :: 29.19.1 Query Profiling Using Performance Schema](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-query-profiling.html)

## explain命令

explain命令可以查看一个DQL语句的执行计划，根据计划可以做出

相应的优化策略。提高执行效率。

例如

```sql
explain select * from emp where empno=7369;
```

执行结果如下

![6fc30cb269e613853ca9103ac206ff33](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/6fc30cb269e613853ca9103ac206ff33.png)

### 各个参数的含义

#### id

id反映出一条select语句执行顺序，id越大优先级越高。id相同则按照自上而下的顺序执行。

这个主要反映在多表查询中

例如

```sql
explain select e.ename,d.dname from emp e join dept d on e.deptno=d.deptno where e.sal=(select sal from emp where ename='ford');
```

结果：

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/8a9e76c6f063dce96adb62082418fe75.png)

e和d表的查询是同级的，而emp是子查询优先级更大。反应的结果就是先进行子查询然后再惊醒e和d表的连接。

### select_type

反映了mysql查询语句的类型。常用值包括：

- SIMPLE：表示查询中不包含子查询或UNION操作。这种查询通常包括一个表或是最多一个联接（JOIN）

- PRIMARY：表示当前查询是一个主查询。（主要的查询）

- UNION：表示查询中包含UNION操作

- SUBQUERY：子查询

- DERIVED：派生表（表示查询语句出现在from后面）

### table

表示查询的哪张表

### type

反映了查询表中数据时的访问类型，常见的值：

1.  NULL：效率最高，一般不可能优化到这个级别，只有查询时没有查询表的时候，访问类型是NULL。例如：select 1;

2.  system：通常访问系统表的时候，访问类型是system。一般也很难优化到这个程序。
3.  const：根据主键或者唯一性索引查询，索引值是常量值时。explain select \* from emp where empno=7369;
4.  eq_ref：根据主键或者唯一性索引查询。索引值不是常量值。
5.  ref：使用了非唯一的索引进行查询。
6.  range：使用了索引，扫描了索引树的一部分。
7.  index：表示用了索引，但是也需要遍历整个索引树。
8.  all：全表扫描

效率最高的是NULL，效率最低的是all，从上到下，从高到低。

### possible_keys

这个查询可能会用到的索引

### key

实际用到的索引

### key_len

反映索引中在查询中使用的列所占的总字节数。

### rows

查询扫描的预估计行数。

### Extra

给出了与查询相关的额外信息和说明。这些额外信息可以帮助我们更好地理解查询执行的过程。

## 索引优化

### 最左前缀原则

什么是最左前缀原则？

在我们使用如何复合索引的时候，如果希望索引起作用，查询条件中必须包含最左边的字段。

例子：

表

```sql
create table t_customer(
    id int primary key auto_increment,
    name varchar(255),
    age int,
    gender char(1),
    email varchar(255)
);
```

添加数据

```sql
insert into t_customer values(null, 'zhangsan', 20, 'M', 'zhangsan@123.com');
insert into t_customer values(null, 'lisi', 22, 'M', 'lisi@123.com');
insert into t_customer values(null, 'wangwu', 18, 'F', 'wangwu@123.com');
insert into t_customer values(null, 'zhaoliu', 22, 'F', 'zhaoliu@123.com');
insert into t_customer values(null, 'jack', 30, 'M', 'jack@123.com');
```

创建复合索引

```sql
create index idx_name_age_gender on t_customer(name,age,gender); #这个name就是最左边的字段
```

查询

```sql
explain select * from t_customer where name='zhangsan' and age=20 and gender='M';
```

这是完全使用的索引，key是索引的名字，key_len=1033表示使用到的索引的长度

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/d34edff3c2bc2e431249d314ff02f3a6.png)

我们试试在查询条件里面不包括name字段会如何

```sql
explain select * from t_customer where gender='M' and age=20;
```

没有使用任何索引

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/edc56a12ef8be9539c01c8c8cae1e33c.png)

我们在试试包括了name，但是没有使用全部复合索引字段的情况

```sql
explain select * from t_customer where name='zhangsan' and age=20;
```

这个使用了索引，但是注意这个key_len=1028，表示这是使用了一部分的索引。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/0f33f1b6c957f011d1f53765b4421c2b.png)

还有只包括name的情况和除了name，其他复合索引的单个字段做条件的情况，大家去试试会发现只要没有name做条件，这个idx_name_age_gender索引就不会起作用。

### 索引失效的情况

- 索引列参加了运算，索引失效

  ```sql
  select * from t_emp where sal*10 > 50000;
  #这个sal作为索引，但是这个索引列参与的运算，那这个索引就不会起作用
  ```

- 索引列进行模糊查询的时候以‘%’开始的，索引失效

  ```sql
  #这个name作为索引
  # 索引有效
  explain select * from t_emp where name like '张%';
  #索引失效
  explain select * from t_emp where name like '%张';
  ```

- 索引列是字符串类型，但查询的时候没有单引号，索引失效

  ```sql
  #age是索引字段，但是是字符类型
  #索引生效
  explain select * from t_emp where age='20';
  #索引失效
  explain select * from t_emp where age=20;
  ```

- 查询条件中有or，只要有未添加索引的字段，索引失效

  ```sql
  #假如name和sal都是索引，那么索引生效，只有它们两个字段有一个字段不是索引字段，那么索引就不会生效
  explain select * from t_emp where name='张三' or sal=5000;
  ```

- 当查询的符合条件的记录在表中的占比很大的时候，不会走索引，因为这是mysql内部优化，mysql认为这种情况不走索引效率更高。

### 指定索引操作

当一个字段上既有单列索引，又有复合索引时，我们可以通过以下的SQL提示来要求该SQL语句执行时采用哪个索引：

- use index(索引名称)：建议使用该索引，只是建议，底层mysql会根据实际效率来考虑是否使用你推荐的索引。

- ignore index(索引名称)：忽略该索引

- force index(索引名称)：强行使用该索引

例子：

一个表

```sql
CREATE TABLE student  (
  sno char(5) NOT NULL,
  sname varchar(20)  NOT NULL,
  sdept varchar(20) NOT NULL,
  sclass char(2),
  ssex char(1),
  birthday date NULL DEFAULT NULL,
  totalcredit decimal(4, 1) NULL DEFAULT NULL,
  PRIMARY KEY (sno) USING BTREE,
  INDEX idx_sname_sdept(sname ASC, sdept ASC) USING BTREE,
  INDEX name_index(sname ASC) USING BTREE
) ;
```

我们执行

```sql
explain select * from student where sname ='李丽';
```

可以看到，我们使用的是idx_sname_sdept这个符合索引，那如何指定它使用单列索引更优。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/4881e329025e41b1396cc79bf88abc3d.png)

我们可以通过在表名后面添加use index(索引名称)来指定查询使用的索引。

```sql
explain select * from student use index(name_index) where sname ='李丽';
```

这个就是使用了name_index索引

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/942c663b26a8bb1afd4bc0455a3a6ba6.png)

忽略索引和强制使用索引不再演示，与上面相同。

### 覆盖索引

覆盖索引指的是查询的字段尽量是索引覆盖的字段，这个可以避免不必要的回表。

回表：

这里涉及到Mysql底层的查询过程，对于一个查询先通过索引（如果是非主键索引）在二级索引树中找到对应的记录的主键值，如果需要查询除了索引字段之外的字段，需要通过这个记录的主键值再通过主键索引查询找到整个记录。

例如在student表中执行

```sql
select * from student use index(name_index) where sname ='李丽';
```

底层就会通过通过sname_index索引找到记录的主键值，然后再通过这个主键值进行查找完整的记录数据。

原理是在非主键索引树中的叶子节点存储的索引列值和主键值，我们要找到完整的数据，就需要再通过主键值（主键的索引树中的叶子节点存放着完整的数据）查找的完整的数据，这步操作就叫回表。

执行下面语句就不会发生回表。

```sql
explain select sname from student use index(name_index) where sname = '李丽';
```

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/bcf8b50fd738ab85783cd21c61053110.png)

结果中Extra的信息是Using index表示没有进行回表操作，只使用索引。

```sql
explain select sname,sdept from student use index(idx_sname_sdept) where sname = '李丽';
```

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/b6990b30c68be784993f22afdf75f2fe.png)

这样同样也是没有进行回表操作

### 前缀索引

如果一个字段的类型是varchar或text，里面存储这长字符串或文本，我们对这个字段建立索引就会消耗大量的空间，这时候我们通常可以截取字符串中前几个字符作为这个字段的索引。这种索引就叫做前缀索引。

例如

```sql
create index idx_sname_2 on student(sname(2));
#表示将sname中前两个字符作为索引值
```

使用前缀索引时，需要通过以下公式来确定使用前几个字符作为索引：

```sql
select count(distinct substring(ename,1,前几个字符)) / count(*) from student;
```

以上查询结果越接近1，表示索引的效果越好。（原理：做索引值的话，索引值越具有唯一性效率越高）

例如

```sql
select count(distinct substring(sname,1,2)) / count(*)  as proportion from student;
+------------+
| proportion |
+------------+
|     0.9091 |
+------------+
```

## SQL语句优化

### order by的优化

在order by之后的字段如果创建了索引将会使用索引，反之则不会

使用explain可以看出来

例如

```sql
explain select sname from student order by sname;
```

Extra中显示Using index表示使用了索引

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/de0b7864517a21ef4c0b59d5668f8bcf.png)

```sql
explain select sclass from student order by sclass;
```

Extra显示是Using filesort表示没有使用索引排序，将数据读到内存中然后在进行排序的。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/fce5e457f7299d39d308a4074d4ac2af.png)

**注意**：这个索引默认是降序排列的

如果是单个字段升序排列也是使用到索引的，但是如果是多个字段的复合排序就不一定会使用索引。

例如

```mysql
explain select sname from student order by sname desc;
```

这样表示从后开始扫描索引树，也是使用了索引

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/02deb7de99aa45487e0bfef3375292e6.png)

```sql
explain select sname,sdept from student order by sname,sdept;
```

这里是先升序排sname再升序排sdept，和创建说索引时的情况相同。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/021411a5e7ead14bbcae8a8d09faa100.png)

但是如果先按照sname降序，sdept升序，那么前面的sname排序会使用索引，后面sdept排序的时候不会使用索引。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/7ccfff4749a2cd53b9889a23b52cceef.png)

解决方式，创建一个按sname降序，sdept升序的索引

```sql
create index ids_sname_sdept on student(sname desc,sdept asc);
```

这样就没有问题了

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/7259dc78fb3ab412aac7774d04d32dfe.png)

### group by的优化

在group by后面的字段如果创建了索引那么也会时候索引优化查询，但是值得注意的是如果这个字段是复合索引的一部分，查询条件需要满足最左前缀原则。

例如

```sql
explain select count(*) , sdept from student group by sdept;
```

显示使用了临时表，效率低，因为sdept不是索引的最左前缀。

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/a35853dc3151138b08195ecbc944ff12.png)

但是我们，加上一个sname字段的条件就会发现，没有使用临时表，效率变高了。

```sql
explain select count(*) , sdept from student where sname='黎明' group by sdept;
```

![img](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/3584917b77348dbf2f1d283c523b6ed2.png)

### 主键优化

主键的设计要点：

- 主键值不要太长，二级索引树的叶子节点中存储的是索引列和主键值，主键值太长会浪费空间
- 主键值尽量不要修改，因为主键值修改，聚集索引一定会重新排序。
- 在插入数据时，主键值最好是顺序插入，不要乱序插入，因为乱序插入可能会导致B+树叶子结点频繁的进行页分裂与页合并操作，效率较低。

### count优化

count(\*)用来统计记录条数是效率最高的

其他方式的确区别

- count(主键)
  - 原理：将每个主键值取出，累加

- count(常量值)
  - 原理：获取到每个常量值，累加

- count(字段)
  - 原理：取出字段的每个值，判断是否为NULL，不为NULL则累加。

- count(\*)
  - 原理：不用取值，底层mysql做了优化，直接统计总行数，效率最高。

## 总结

ame字段的条件就会发现，没有使用临时表，效率变高了。

```sql
explain select count(*) , sdept from student where sname='黎明' group by sdept;
```

[外链图片转存中...(img-5lQeMnId-1768468710604)]

### 主键优化

主键的设计要点：

- 主键值不要太长，二级索引树的叶子节点中存储的是索引列和主键值，主键值太长会浪费空间
- 主键值尽量不要修改，因为主键值修改，聚集索引一定会重新排序。
- 在插入数据时，主键值最好是顺序插入，不要乱序插入，因为乱序插入可能会导致B+树叶子结点频繁的进行页分裂与页合并操作，效率较低。

### count优化

count(\*)用来统计记录条数是效率最高的

其他方式的确区别

- count(主键)
  - 原理：将每个主键值取出，累加

- count(常量值)
  - 原理：获取到每个常量值，累加

- count(字段)
  - 原理：取出字段的每个值，判断是否为NULL，不为NULL则累加。

- count(\*)
  - 原理：不用取值，底层mysql做了优化，直接统计总行数，效率最高。

## 总结

关于mysql的底层优化，其实主要是对索引的应用，而索引树底层实现是B+树，如果理解和B+这个数据结构，那么索引也就掌握了大半，此外还有一些细节方面，本文章并没有涉及，如果要更深入理解Mysql的优化，阅读官方文档和实践是不错的方法。
