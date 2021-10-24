#                            Hive资料

## 一 .hive概念

```sql
一.什么是Hive
Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能
 id     name   salary
 int    string  int
1001    张三    10000
1002    李四    20000
1003    王五	  30000
...../hive/salary.txt

-----------------------------
跟表 差  表名 salary 
id(int) name(stirng)   salary(int)
1001    张三            10000
1002    李四            20000
1003    王五	          30000
-----------------------------
number       up    down
1388888888  5879  13440
1366666666  5139  13001
1311111111  5453  13445
```

```
二.hive和mr的关系
1.hive封装很多mr的模板，代替了写mr
2.hive的数据存储在hdfs上
3.执行需要yarn
```

```sql
三.hive的优缺点
1.优点:上手简单，海量数据分析和计算，自定义函数，可以完成用户自定义化的需求
2.缺点：机翻，不够智能，粒度较粗，调优困难.mr的缺点
```

```sql
四.hive的架构原理
1.hive的客户端 cli jdbc
2.driver(驱动器)
  <1>.解析器：判断sql语法的问题
  <2>.编译器：将写好的sql制定执行计划
  <3>.优化器：将制定好的执行计划进行优化
  <4>.执行器：找mr模板执行sql
3.hadoop 
4.meta store 元数据存储(描述数据的数据)
```

```sql
五.hive的运行机制
```

```sql
六.hive的参数配置
hive-default   <   hive-site  <   hive -hiveconf    < hive  set   优先级依次增大

hive参数配置的四种方式 : hive-default  hive-site 永久生效  hive -hiveconf   hive  set 临时生效 只对单此会话有作用
```

## DDL

#### 一.库的ddl

```sql
CREATE DATABASE [IF NOT EXISTS] database_name
[COMMENT database_comment]   --这个数据库的解释(写一下这个数据库是拿来干什么的)
[LOCATION hdfs_path]
[WITH DBPROPERTIES (property_name=property_value, ...)]; --关于库的属性值  非常鸡肋


create database db_hive
comment 'this is my first db'
with dbproperties('type'='ddd','user'='atguigu');

create database db_hive2
location '/dsadads';

create database if not exists db_hive2;

alter database db_hive set dbproperties('type'='db');

alter database db_hive set dbproperties('createtime'='2020-07-31');

drop database 数据库名;  --删除

drop database 数据库名 cascade;  --强制删除 如果里面存在表 一起全部删除 
```

```html
关于hive的元数据
https://blog.csdn.net/xjp8587/article/details/81411879   
```

```sql
表的ddl
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name   --EXTERNAL表示创建表是否是外部表
[(col_name data_type [COMMENT col_comment], ...)]    --表的字段名 和字段类型  ，以及对字段的解释
[COMMENT table_comment]                              --对于表的解释
[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]   --分区
[CLUSTERED BY (col_name, col_name, ...)                            --分桶
[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]   --分几个桶
[ROW FORMAT DELIMITED                        --对于当前的表分隔的说明
 [FIELDS TERMINATED BY char]                 --对于字段的分隔说明  --默认是ascii 码表的001 ^A
 yangyang,caicai_susu, xiao yang:18_xiaoxiao yang:19,chao yang_beijing_10011 
                                             --对于此行数据的分隔类型是 ','
 [COLLECTION ITEMS TERMINATED BY char]       --对于集合的分隔说明  --默认是ascii 码表的002 ^B
 caicai_susu xiao yang:18_xiaoxiao yang:19 chao yang_beijing_10011
                                             --对于三个集合 map array struct 分隔是'_'
 [MAP KEYS TERMINATED BY char]               --对于map的kv分隔类型 --默认是ascii 码表的003 ^C
 xiao yang:18 
                                             --对于当前map集合分隔符是 ':'
 [LINES TERMINATED BY char]                  --对于每行数据而言的分隔类型  '\n'
] 
[STORED AS file_format]                      --对于当前表对应文件存储类型
[LOCATION hdfs_path]                         --指定的hdfs路径 如果不指定默认在库里面
[TBLPROPERTIES (property_name=property_value, ...)]  --表的属性
[AS select_statement]                        --根据查询的结果创建表(包括表的结构和数据)
[LIKE table_name]                            --模仿一张表(只有表的结构没有数据)
```

```sql
一管理表(内部表)
  --hive掌握数据生命周期(会伴随着表的删除而删除)  -测试，中间表
create table student (
id int,
name string
)
row format delimited fields terminated by '\t'

---------根据结果创建表
create table student2 as select * from student;  --表结构和数据 但是 不带分隔符
---------模仿一张表
create table student3 like student;             --表结构并且 带分隔符
---------展示表
show tables;
---------描述表
desc table_name;
---------详情描述表
desc formatted table_name;
```

```sql
一 外部表          --除了上述内部表的两种情形 全部都是外部表
 -- hive没有掌握数据声明周期(伴随着表的删除不会删除表里的数据，只会删除表的元数据)  
 create external table if not exists dept(
deptno int,
dname string,
loc int
)
row format delimited fields terminated by '\t'
location '/company/dept';


create external table if not exists emp(
empno int,
ename string,
job string,
mgr int,
hiredate string, 
sal double, 
comm double,
deptno int)
row format delimited fields  terminated by '\t'
location '/company/emp';
```

```sql
关于内部表和外部表的相互转化
Table Type: MANAGED_TABLE                             
Table Parameters:         
     EXTERNAL                 FALSE    
关于表示是否是内部表还是外部表 通过的就是tblproperties 里面的 EXTERNAL属性来决定的.

```

```sql
---------修改表
 --修改表名
 ALTER TABLE table_name RENAME TO new_table_name
 
 alter table student2 rename to student4;
 
 --修改列
 ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type [COMMENT col_comment] [FIRST|AFTER column_name]
 
 create table student(id tinyint, name string) row format delimited fields terminated by '\t';
 
 alter table student change id id tinyint;
 --增加和替换列

 ALTER TABLE table_name ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...) 
 
 alter table student add columns(createtime string);
 
 -- 对于replace而言
 -- 如果你对列增加了 那么增加的列的类型是可以任意类型
 -- 如果你对列减少了 那么剩下的列跟之前的列应该满足类型上 1:1对应的关系(满足类型从小到大)
alter table student replace columns(idss int , namess string );
alter table student replace columns(idss int , namess string , type int , createtime  string );
```

## DML

```sql
数据导入  
load data [local] inpath '数据的path' [overwrite] into table student [partition (partcol1=val1,…)];

create table stu1(id int , name string) row format delimited fields terminated by '\t

create table stu2(id int , name string) row format delimited fields terminated by '\t'

load data local inpath '/opt/module/hive/datas/stu1.txt' into table stu1;

load data  inpath '/datas/stu2.txt' into table stu2;

--如果是从本地 local 那么是复制进去  如果不是本地  那么是剪切进去的

load data local inpath '/opt/module/hive/datas/stu1.txt' overwrite into table stu1;

------------------------------------------------------------
insert 

insert into stu1 values(1,'yizuzhang'),(2,'erzuzhang');
--  通过查询插入
insert overwrite table stu1 select * from stu2;

insert overwrite table student select * from stu2;

----------------------------------------------
location

create table teacher (
id int,
name string
)
row format delimited fields terminated by '\t'
location '/teacher';
```

```sql
数据导出
----导出到本地
insert overwrite local directory '/opt/module/hive/datas/export1' select * from stu1;

insert overwrite local directory '/opt/module/hive/datas/export1' 
  row format delimited fields terminated by '\t' select * from stu1;

insert overwrite directory '/export1' 
  row format delimited fields terminated by '\t' select * from stu1;
  
----export 导出
export table teacher to '/teacher';

-- import 导入
import table teacher2 from '/teacher';

import 导入必须是export 导出

```

## 查询

```sql
SELECT [ALL | DISTINCT] select_expr, select_expr, ...  
  FROM table_reference
  [WHERE where_condition]
  [GROUP BY col_list]
  [HAVING col_list]
  [ORDER BY col_list]
  [CLUSTER BY col_list
  | [DISTRIBUTE BY col_list] [SORT BY col_list]
  ]
 [LIMIT number]

create table if not exists dept(
deptno int,   --部门编号
dname string, --部门名称
loc int       --部门地址标号
)
row format delimited fields terminated by '\t';

create table if not exists emp(
empno int,     --员工编号
ename string,  --员工名称
job string,    --员工职位
mgr int,       --员工的领导
hiredate string, --员工的入职日期
sal double,      --员工的薪资
comm double,     --员工的奖金
deptno int)      --员工的部门id
row format delimited fields terminated by '\t'

------名字开头待s
select * from emp where ename like 'S%';
select * from emp where ename rlike '^S';
-----名字末尾带s
select * from emp where ename like '%S';
select * from emp where ename rlike 'S$';
-----名字中带s
select * from emp where ename like '%S%';
select * from emp where ename rlike '[S]';
```

```sql
join

--员工表和部门表中的部门编号相等，查询员工编号、员工名称和部门名称
select
e.empno,
e.ename,
d.deptno,
d.dname
from emp e join dept d
on e.deptno !=d.deptno;
--合并员工表
select e.*,d.dname,d.loc from emp e join dept d on e.deptno=d.deptno;

--左连接  将emp作为主表
select
e.empno,
e.ename,
e.deptno,
d.deptno,
d.dname
from emp e left join dept d
on e.deptno=d.deptno;

--右连接  将emp作为主表
select
e.empno,
e.ename,
d.deptno,
d.dname
from dept d right join emp e
on e.deptno=d.deptno;

--满外连接  
select
e.empno,
e.ename,
d.deptno,
d.dname
from emp e full join dept d
on e.deptno =d.deptno
--在mysql如果想要实现满外连接的方式
select
e.empno,
e.ename,
d.deptno,
d.dname
from dept d left join emp e
on e.deptno=d.deptno

union all

select
e.empno,
e.ename,
d.deptno,
d.dname
from dept d right join emp e
on e.deptno=d.deptno;

--多表联查
---查到 员工名称，员工部门名称，员工部门位置名称
--第一种写法
select
e.ename,
d.dname,
l.loc_name
from emp e join dept d
on e.deptno=d.deptno
join location l
on d.loc=l.loc
--第二种写法
select
e.ename,
d.dname,
l.loc_name
from emp e join dept d join location l
on e.deptno=d.deptno and d.loc=l.loc
--笛卡尔积
  连接条件失效
  连接条件衡成立
select * from dept,emp;
select * from dept join emp;
select * from dept join emp on 1=1;

```

```sql
全局排序  order
-- 按照工资给emp所有的人做一个全局排序
select empno,ename,sal,deptno from emp order by sal asc --升序排序
select empno,ename,sal,deptno from emp order by sal desc --倒序排序
--按照不同部门里所有人工资排序
select empno,ename,sal,deptno from emp order by deptno,sal;
select empno,ename,sal,deptno from emp order by deptno desc,sal desc;
```

```sql
sort by 
set mapreduce.job.reduces=3; --其实设置当前的 分区数 =》3

sort by 对于每个reduce而言是 有序,但是在设置分区数过后 他是随机进入分区的 --没有人会单独使用

distribute by  结合着 sort by 使用

select
empno,
ename,
sal,
deptno
from emp 
distribute by deptno sort by sal;

distribute by 以什么分区   sort by 分区内区内排序  需要手动这是map的分区数

cluster by  分区排序

select * from emp cluster by deptno asc;
以下的sql等同于上面的sql
select * from emp  distribute by deptno sort by sal;
```

## 分区和分桶

#### 分区表

```sql
为什么要分区表
 --首先在hive里面没有这个索引的概念,所以每次查询会暴力扫描整张表
 --加了分区的概念(分区分的就是表下面的文件夹)
  创建一个分区表
create table dept_partition1(
deptno int, dname string, loc string
)
partitioned by (deptno int)  --这个分区字段是个伪列(这个列不存在建表语句里面)
row format delimited fields terminated by '\t';
load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table dept_partition partition(day='20200401');

--联合查询
select * from dept_partition where day='20200401'
union
select * from dept_partition where day='20200402'
union
select * from dept_partition where day='20200403';

--查看表里面的所有分区
 show partitions dept_partition;
 
 --删除分区
 alter table dept_partition drop partition(day='__HIVE_DEFAULT_PARTITION__');
 --删除多个分区
 alter table dept_partition drop partition(day='20200403'),partition(day='20200402');
 --增加分区 
 alter table dept_partition add partition(day='20200402');
 --增加多个分区
 alter table dept_partition add partition(day='20200403') partition(day='20200404');
 
 --查询表的分区
 desc dept_partition;
 ---------二级分区
load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table dept_partition2 partition(day='20200401',hour='12');

load data local inpath '/opt/module/hive/datas/dept_20200402.log' into table dept_partition2 partition(day='20200401',hour='14');

load data local inpath '/opt/module/hive/datas/dept_20200403.log' into table dept_partition2 ;

select * from dept_partition2 where hour = '12';
--删除二级分区
alter table dept_partition2 drop partition(day='20200401',hour='12');
alter table dept_partition2 drop partition(day='20200401',hour='13');

--数据与分区表产生关联的三种方式(去增加元数据)
--1 修复分区
  msck repair table dept_partition2;

--2 增加分区 
 alter table dept_partition2 add partition(day='20200402',hour='15');
--3 load数据
load data local inpath '/opt/module/hive/datas/dept_20200401.log' into table
 dept_partition2 partition(day='20200402',hour='16');


--动态分区

load data local inpath '/opt/module/hive/datas/dept.txt' into table dept_partition_dy;

load data local inpath '/opt/module/hive/datas/dept_20200501.log' into table dept_partition;

load data local inpath '/opt/module/hive/datas/dept_20200601_2level.log' into table dept_partition2;

create table dept_partition3(id int) partitioned by (name string ,loc int) row format delimited fields terminated by '\t';

insert into table dept_partition4 partition(name,loc) select deptno,dname, loc from dept;

create table dept_partition4(id int) partitioned by (name string ,loc int) row format delimited fields terminated by '\t';

```

#### 分桶

```sql
分桶的是数据
create table stu_buck(id int, name string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';

-----创建一个既分区有分桶的表
create table stu_buck_part(id int, name string)
partitioned by (day string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';

load data inpath '/student/student.txt' into table stu_buck_part partition(day ='20200801');
```

## 函数

#### 窗口函数

```sql
1.窗口函数是什么
mysq 5.7，5.6 是没有窗口函数  但是mysql 8.0有窗口函数

oracle  一直有窗口函数

hive里面也支持窗口函数

窗口函数是高级函数
以下函数是窗口函数
LEAD LEAD(col,n, default_val)：往后第n行数据  col是字段 n表示第几行,default_val 如果没有往后第n行则用默认值代替 
LAG LAG(col,n,default_val)：往前第n行数据   col是字段 n表示第几行,default_val 如果没有往前第n行则用默认值代替
FIRST_VALUE FIRST_VALUE (col,true/false)：当前窗口下的第一个值，第二个参数为true，跳过空值
LAST_VALUE LAST_VALUE (col,true/false)：当前窗口下的最后一个值，第二个参数为true，跳过空值
--标准的聚合函数
COUNT
SUM
MIN
MAX
AVG
--排名和分析函数
RANK
ROW_NUMBER
DENSE_RANK
NTILE
窗口函数=函数+窗口
窗口--可以限定函数计算的范围

窗口函数的语法
窗口函数() over([partition by 字段...] [order by  字段..] [窗口子句])
窗口函数本身的执行顺序
1.over ()最大的窗口范围
2.partition by 表示对over划分窗口再次划分
3.order by 对 over/partition 的窗口数据按照字段做排序  asc desc
4.窗口子句 对于over/partition 内的数据 给定函数的运算范围
5.窗口函数()  对每一行数据做 窗口范围内的运算

over 是表示最大的窗口范围

partition by 字段... 按照字段是否相同来划分更细的窗口 将字段相同的数据扔到同一个细窗口里面

窗口子句  当有只有over的时候窗口的范围是over指定的范围 当有partition的时候 窗口子句范围是, partition by 过后所有细窗口范围，他们之间相互独立

并不是所有行数都需要窗口子句
Rank, NTile, DenseRank,ROW_NUMBER ,Lead, Lag .

当有order by 但是没有窗口子句的时候默认子句范围 the WINDOW specification defaults to rows BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW.

当order by 和 窗口子句都没有的时候子句的默认范围 , the WINDOW specification defaults to ROW BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING.

窗口子句
(ROWS | RANGE) BETWEEN (UNBOUNDED | [num]) PRECEDING AND ([num] PRECEDING | CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
(ROWS | RANGE) BETWEEN CURRENT ROW AND (CURRENT ROW | (UNBOUNDED | [num]) FOLLOWING)
(ROWS | RANGE) BETWEEN [num] FOLLOWING AND (UNBOUNDED | [num]) FOLLOWING

```

##### 需求1  查询在2017年4月份购买过的顾客及总人数 总人次

```sql
--按照4月份过滤
select *
from business
where month(orderdate)=4;  

select *
from business
where substring(orderdate,1,7)='2017-04'


select *
from business
where date_format(orderdate,'yyyy-MM')='2017-04';

--求总人数窗口函数1
select 
name,
count(*)over(rows between UNBOUNDED PRECEDING and UNBOUNDED FOLLOWING)
from business
where month(orderdate)=4
group by name;
+-------+-----------------+
| name  | count_window_0  |
+-------+-----------------+
| mart  | 2               |
| jack  | 2               |
+-------+-----------------+
--求总人数窗口函数2
select name,count(*) over () 
from business 
where substring(orderdate,1,7) = '2017-04' 
group by name;

--需求1变种1  累加统计4月份有多少人消费
select 
name,
count(*)over(rows between UNBOUNDED PRECEDING and current row)
from business
where month(orderdate)=4
group by name;

| name  | count_window_0  |
+-------+-----------------+
| mart  | 1               |
| jack  | 2               |


--需求1变种2  累加统计有多少人消费 
select 
name,
count(*)over(rows between UNBOUNDED PRECEDING and current row)
from business;
+-------+-----------------+
| name  | count_window_0  |
+-------+-----------------+
| mart  | 1               |
| neil  | 2               |
| mart  | 3               |
| neil  | 4               |
| mart  | 5               |
| mart  | 6               |
| jack  | 7               |
| tony  | 8               |
| jack  | 9               |
| jack  | 10              |
| tony  | 11              |
| jack  | 12              |
| tony  | 13              |
| jack  | 14              |
+-------+-----------------+

--需求1变种3  统计一共有多少人消费
select 
name,
count(*)over(rows between UNBOUNDED PRECEDING and UNBOUNDED FOLLOWING)
from business;
+-------+-----------------+
| name  | count_window_0  |
+-------+-----------------+
| mart  | 14              |
| neil  | 14              |
| mart  | 14              |
| neil  | 14              |
| mart  | 14              |
| mart  | 14              |
| jack  | 14              |
| tony  | 14              |
| jack  | 14              |
| jack  | 14              |
| tony  | 14              |
| jack  | 14              |
| tony  | 14              |
| jack  | 14              |
+-------+-----------------+

--用一般函数怎么做(求所有不同月份的总人数和人)
select
name,
date_format(orderdate,'yyyy-MM') orderdate,
cost
from business t1

select
collect_set(name) c_s_n,
t1.orderdate
from (
     select
     name,
     date_format(orderdate,'yyyy-MM') orderdate,
     cost
     from business)t1
group by t1.orderdate  t2

select
t2.c_s_n,
size(t2.c_s_n) p_n,
t2.orderdate
from (select
collect_set(name) c_s_n,
t1.orderdate
from (
     select
     name,
     date_format(orderdate,'yyyy-MM') orderdate,
     cost
     from business)t1
group by t1.orderdate)t2

+------------------+------+---------------+
|     t2.c_s_n     | p_n  | t2.orderdate  |
+------------------+------+---------------+
| ["jack","tony"]  | 2    | 2017-01       |
| ["jack"]         | 1    | 2017-02       |
| ["jack","mart"]  | 2    | 2017-04       |
| ["neil"]         | 1    | 2017-05       |
| ["neil"]         | 1    | 2017-06       |
+------------------+------+---------------+
```

##### 询顾客的购买明细及月购买总额

```sql
select
name,
orderdate,
cost,
sum(cost)over(partition by name,month(orderdate) rows between UNBOUNDED PRECEDING and UNBOUNDED FOLLOWING)
from business;
+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-05  | 46    | 111           |
| jack  | 2017-01-08  | 55    | 111           |
| jack  | 2017-01-01  | 10    | 111           |
| jack  | 2017-02-03  | 23    | 23            |
| jack  | 2017-04-06  | 42    | 42            |
| mart  | 2017-04-13  | 94    | 299           |
| mart  | 2017-04-11  | 75    | 299           |
| mart  | 2017-04-09  | 68    | 299           |
| mart  | 2017-04-08  | 62    | 299           |
| neil  | 2017-05-10  | 12    | 12            |
| neil  | 2017-06-12  | 80    | 80            |
| tony  | 2017-01-04  | 29    | 94            |
| tony  | 2017-01-02  | 15    | 94            |
| tony  | 2017-01-07  | 50    | 94            |
+-------+-------------+-------+---------------+

需求2的变种

select
name,
orderdate,
cost,
sum(cost)over(partition by name,month(orderdate))  --满足order 子句的第二种情况 默认是上无边界到当前行
from business;
+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-05  | 46    | 111           |
| jack  | 2017-01-08  | 55    | 111           |
| jack  | 2017-01-01  | 10    | 111           |
| jack  | 2017-02-03  | 23    | 23            |
| jack  | 2017-04-06  | 42    | 42            |
| mart  | 2017-04-13  | 94    | 299           |
| mart  | 2017-04-11  | 75    | 299           |
| mart  | 2017-04-09  | 68    | 299           |
| mart  | 2017-04-08  | 62    | 299           |
| neil  | 2017-05-10  | 12    | 12            |
| neil  | 2017-06-12  | 80    | 80            |
| tony  | 2017-01-04  | 29    | 94            |
| tony  | 2017-01-02  | 15    | 94            |
| tony  | 2017-01-07  | 50    | 94            |
+-------+-------------+-------+---------------+
需求2的变种  统计每个人一共消费了多少和明细
select
name,
orderdate,
cost,
sum(cost)over(partition by name)
from business;

 name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-05  | 46    | 176           |
| jack  | 2017-01-08  | 55    | 176           |
| jack  | 2017-01-01  | 10    | 176           |
| jack  | 2017-04-06  | 42    | 176           |
| jack  | 2017-02-03  | 23    | 176           |
| mart  | 2017-04-13  | 94    | 299           |
| mart  | 2017-04-11  | 75    | 299           |
| mart  | 2017-04-09  | 68    | 299           |
| mart  | 2017-04-08  | 62    | 299           |
| neil  | 2017-05-10  | 12    | 92            |
| neil  | 2017-06-12  | 80    | 92            |
| tony  | 2017-01-04  | 29    | 94            |
| tony  | 2017-01-02  | 15    | 94            |
| tony  | 2017-01-07  | 50    | 94            |
+-------+-------------+-------+---------------+
需求2的变种 累计每个人一共消费了多少和明细
select
name,
orderdate,
cost,
sum(cost)over(partition by name rows between UNBOUNDED PRECEDING and current row)
from business;

+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-05  | 46    | 46            |
| jack  | 2017-01-08  | 55    | 101           |
| jack  | 2017-01-01  | 10    | 111           |
| jack  | 2017-04-06  | 42    | 153           |
| jack  | 2017-02-03  | 23    | 176           |
| mart  | 2017-04-13  | 94    | 94            |
| mart  | 2017-04-11  | 75    | 169           |
| mart  | 2017-04-09  | 68    | 237           |
| mart  | 2017-04-08  | 62    | 299           |
| neil  | 2017-05-10  | 12    | 12            |
| neil  | 2017-06-12  | 80    | 92            |
| tony  | 2017-01-04  | 29    | 29            |
| tony  | 2017-01-02  | 15    | 44            |
| tony  | 2017-01-07  | 50    | 94            |
+-------+-------------+-------+---------------+

```

##### 上述的场景, 将每个顾客的cost按照日期进行累加

```sql
select
name,
orderdate,
cost,
sum(cost)over(partition by name order by orderdate)
from business;

+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-01  | 10    | 10            |
| jack  | 2017-01-05  | 46    | 56            |
| jack  | 2017-01-08  | 55    | 111           |
| jack  | 2017-02-03  | 23    | 134           |
| jack  | 2017-04-06  | 42    | 176           |
| mart  | 2017-04-08  | 62    | 62            |
| mart  | 2017-04-09  | 68    | 130           |
| mart  | 2017-04-11  | 75    | 205           |
| mart  | 2017-04-13  | 94    | 299           |
| neil  | 2017-05-10  | 12    | 12            |
| neil  | 2017-06-12  | 80    | 92            |
| tony  | 2017-01-02  | 15    | 15            |
| tony  | 2017-01-04  | 29    | 44            |
| tony  | 2017-01-07  | 50    | 94            |

需求变种1   统计当天消费和上一次消费的和

select
name,
orderdate,
cost,
sum(cost)over(partition by name order by orderdate rows between 1 PRECEDING and current row )
from business;

+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-01  | 10    | 10            |
| jack  | 2017-01-05  | 46    | 56            |
| jack  | 2017-01-08  | 55    | 101           |
| jack  | 2017-02-03  | 23    | 78            |
| jack  | 2017-04-06  | 42    | 65            |
| mart  | 2017-04-08  | 62    | 62            |
| mart  | 2017-04-09  | 68    | 130           |
| mart  | 2017-04-11  | 75    | 143           |
| mart  | 2017-04-13  | 94    | 169           |
| neil  | 2017-05-10  | 12    | 12            |
| neil  | 2017-06-12  | 80    | 92            |
| tony  | 2017-01-02  | 15    | 15            |
| tony  | 2017-01-04  | 29    | 44            |
| tony  | 2017-01-07  | 50    | 79            |

需求变种2   统计当天消费和下一次消费的和
select
name,
orderdate,
cost,
sum(cost)over(partition by name order by orderdate rows between current row and 1 following )
from business;

+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-01  | 10    | 56            |
| jack  | 2017-01-05  | 46    | 101           |
| jack  | 2017-01-08  | 55    | 78            |
| jack  | 2017-02-03  | 23    | 65            |
| jack  | 2017-04-06  | 42    | 42            |
| mart  | 2017-04-08  | 62    | 130           |
| mart  | 2017-04-09  | 68    | 143           |
| mart  | 2017-04-11  | 75    | 169           |
| mart  | 2017-04-13  | 94    | 94            |
| neil  | 2017-05-10  | 12    | 92            |
| neil  | 2017-06-12  | 80    | 80            |
| tony  | 2017-01-02  | 15    | 44            |
| tony  | 2017-01-04  | 29    | 79            |
| tony  | 2017-01-07  | 50    | 50            |
+-------+-------------+-------+---------------+
需求变种3   统计上一次消费到下一次消费的和

select
name,
orderdate,
cost,
sum(cost)over(partition by name order by orderdate rows between 1 PRECEDING and 1 following )
from business;

+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-01  | 10    | 56            |
| jack  | 2017-01-05  | 46    | 111           |
| jack  | 2017-01-08  | 55    | 124           |
| jack  | 2017-02-03  | 23    | 120           |
| jack  | 2017-04-06  | 42    | 65            |
| mart  | 2017-04-08  | 62    | 130           |
| mart  | 2017-04-09  | 68    | 205           |
| mart  | 2017-04-11  | 75    | 237           |
| mart  | 2017-04-13  | 94    | 169           |
| neil  | 2017-05-10  | 12    | 92            |
| neil  | 2017-06-12  | 80    | 92            |
| tony  | 2017-01-02  | 15    | 44            |
| tony  | 2017-01-04  | 29    | 94            |
| tony  | 2017-01-07  | 50    | 79            |

需求变种4 直接按日期累加消费并且需要明细
select
name,
orderdate,
cost,
sum(cost)over(order by orderdate)
from business;
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-01  | 10    | 10            |
| tony  | 2017-01-02  | 15    | 25            |
| tony  | 2017-01-04  | 29    | 54            |
| jack  | 2017-01-05  | 46    | 100           |
| tony  | 2017-01-07  | 50    | 150           |
| jack  | 2017-01-08  | 55    | 205           |
| jack  | 2017-02-03  | 23    | 228           |
| jack  | 2017-04-06  | 42    | 270           |
| mart  | 2017-04-08  | 62    | 332           |
| mart  | 2017-04-09  | 68    | 400           |
| mart  | 2017-04-11  | 75    | 475           |
| mart  | 2017-04-13  | 94    | 569           |
| neil  | 2017-05-10  | 12    | 581           |
| neil  | 2017-06-12  | 80    | 661           |

```

##### 查询顾客购买明细以及上次的购买时间和下次购买时间

```sql
select
name,
orderdate,
cost,
lag(orderdate)over(partition by name order by orderdate) prev_time,
lead(orderdate)over(partition by name order by orderdate) next_time
from business;

+-------+-------------+-------+-------------+-------------+
| name  |  orderdate  | cost  |  prev_time  |  next_time  |
+-------+-------------+-------+-------------+-------------+
| jack  | 2017-01-01  | 10    | NULL        | 2017-01-05  |
| jack  | 2017-01-05  | 46    | 2017-01-01  | 2017-01-08  |
| jack  | 2017-01-08  | 55    | 2017-01-05  | 2017-02-03  |
| jack  | 2017-02-03  | 23    | 2017-01-08  | 2017-04-06  |
| jack  | 2017-04-06  | 42    | 2017-02-03  | NULL        |
| mart  | 2017-04-08  | 62    | NULL        | 2017-04-09  |
| mart  | 2017-04-09  | 68    | 2017-04-08  | 2017-04-11  |
| mart  | 2017-04-11  | 75    | 2017-04-09  | 2017-04-13  |
| mart  | 2017-04-13  | 94    | 2017-04-11  | NULL        |
| neil  | 2017-05-10  | 12    | NULL        | 2017-06-12  |
| neil  | 2017-06-12  | 80    | 2017-05-10  | NULL        |
| tony  | 2017-01-02  | 15    | NULL        | 2017-01-04  |
| tony  | 2017-01-04  | 29    | 2017-01-02  | 2017-01-07  |
| tony  | 2017-01-07  | 50    | 2017-01-04  | NULL        |
+-------+-------------+-------+-------------+-------------+

select
name,
orderdate,
cost,
lag(orderdate,1,'0000-00-00')over(partition by name order by orderdate) prev_time,
lead(orderdate,1,'9999-99-99')over(partition by name order by orderdate) next_time
from business;
+-------+-------------+-------+-------------+-------------+
| name  |  orderdate  | cost  |  prev_time  |  next_time  |
+-------+-------------+-------+-------------+-------------+
| jack  | 2017-01-01  | 10    | 0000-00-00  | 2017-01-05  |
| jack  | 2017-01-05  | 46    | 2017-01-01  | 2017-01-08  |
| jack  | 2017-01-08  | 55    | 2017-01-05  | 2017-02-03  |
| jack  | 2017-02-03  | 23    | 2017-01-08  | 2017-04-06  |
| jack  | 2017-04-06  | 42    | 2017-02-03  | 9999-99-99  |
| mart  | 2017-04-08  | 62    | 0000-00-00  | 2017-04-09  |
| mart  | 2017-04-09  | 68    | 2017-04-08  | 2017-04-11  |
| mart  | 2017-04-11  | 75    | 2017-04-09  | 2017-04-13  |
| mart  | 2017-04-13  | 94    | 2017-04-11  | 9999-99-99  |
| neil  | 2017-05-10  | 12    | 0000-00-00  | 2017-06-12  |
| neil  | 2017-06-12  | 80    | 2017-05-10  | 9999-99-99  |
| tony  | 2017-01-02  | 15    | 0000-00-00  | 2017-01-04  |
| tony  | 2017-01-04  | 29    | 2017-01-02  | 2017-01-07  |
| tony  | 2017-01-07  | 50    | 2017-01-04  | 9999-99-99  |
+-------+-------------+-------+-------------+-------------+

```

##### 查询顾客每个月第一次的购买时间 和 每个月的最后一次购买时间

```sql
select
name,
orderdate,
cost,
first_value(orderdate)over(partition by name,month(orderdate) order by orderdate 
rows between UNBOUNDED PRECEDING and UNBOUNDED following ) first_date,
last_value(orderdate)over(partition by name,month(orderdate) order by orderdate
rows between UNBOUNDED PRECEDING and UNBOUNDED following ) last_date
from business

+-------+-------------+-------+-------------+-------------+
| name  |  orderdate  | cost  | first_date  |  last_date  |
+-------+-------------+-------+-------------+-------------+
| jack  | 2017-01-01  | 10    | 2017-01-01  | 2017-01-08  |
| jack  | 2017-01-05  | 46    | 2017-01-01  | 2017-01-08  |
| jack  | 2017-01-08  | 55    | 2017-01-01  | 2017-01-08  |
| jack  | 2017-02-03  | 23    | 2017-02-03  | 2017-02-03  |
| jack  | 2017-04-06  | 42    | 2017-04-06  | 2017-04-06  |
| mart  | 2017-04-08  | 62    | 2017-04-08  | 2017-04-13  |
| mart  | 2017-04-09  | 68    | 2017-04-08  | 2017-04-13  |
| mart  | 2017-04-11  | 75    | 2017-04-08  | 2017-04-13  |
| mart  | 2017-04-13  | 94    | 2017-04-08  | 2017-04-13  |
| neil  | 2017-05-10  | 12    | 2017-05-10  | 2017-05-10  |
| neil  | 2017-06-12  | 80    | 2017-06-12  | 2017-06-12  |
| tony  | 2017-01-02  | 15    | 2017-01-02  | 2017-01-07  |
| tony  | 2017-01-04  | 29    | 2017-01-02  | 2017-01-07  |
| tony  | 2017-01-07  | 50    | 2017-01-02  | 2017-01-07  |
+-------+-------------+-------+-------------+-------------+
需求变种1 查询顾客第一次的购买时间 和 最后一次购买时间
select
name,
orderdate,
cost,
first_value(orderdate)over(partition by name order by orderdate 
rows between UNBOUNDED PRECEDING and UNBOUNDED following ) first_date,
last_value(orderdate)over(partition by name order by orderdate
rows between UNBOUNDED PRECEDING and UNBOUNDED following ) last_date
from business
+-------+-------------+-------+-------------+-------------+
| name  |  orderdate  | cost  | first_date  |  last_date  |
+-------+-------------+-------+-------------+-------------+
| jack  | 2017-01-01  | 10    | 2017-01-01  | 2017-04-06  |
| jack  | 2017-01-05  | 46    | 2017-01-01  | 2017-04-06  |
| jack  | 2017-01-08  | 55    | 2017-01-01  | 2017-04-06  |
| jack  | 2017-02-03  | 23    | 2017-01-01  | 2017-04-06  |
| jack  | 2017-04-06  | 42    | 2017-01-01  | 2017-04-06  |
| mart  | 2017-04-08  | 62    | 2017-04-08  | 2017-04-13  |
| mart  | 2017-04-09  | 68    | 2017-04-08  | 2017-04-13  |
| mart  | 2017-04-11  | 75    | 2017-04-08  | 2017-04-13  |
| mart  | 2017-04-13  | 94    | 2017-04-08  | 2017-04-13  |
| neil  | 2017-05-10  | 12    | 2017-05-10  | 2017-06-12  |
| neil  | 2017-06-12  | 80    | 2017-05-10  | 2017-06-12  |
| tony  | 2017-01-02  | 15    | 2017-01-02  | 2017-01-07  |
| tony  | 2017-01-04  | 29    | 2017-01-02  | 2017-01-07  |
| tony  | 2017-01-07  | 50    | 2017-01-02  | 2017-01-07  |

```

```sql
select
name,
orderdate,
cost,
sum(cost)over(order by month(orderdate))
from business
+-------+-------------+-------+---------------+
| name  |  orderdate  | cost  | sum_window_0  |
+-------+-------------+-------+---------------+
| jack  | 2017-01-01  | 10    | 205           |
| jack  | 2017-01-08  | 55    | 205           |
| tony  | 2017-01-07  | 50    | 205           |
| jack  | 2017-01-05  | 46    | 205           |
| tony  | 2017-01-04  | 29    | 205           |
| tony  | 2017-01-02  | 15    | 205           |
| jack  | 2017-02-03  | 23    | 228           |
| mart  | 2017-04-13  | 94    | 569           |
| jack  | 2017-04-06  | 42    | 569           |
| mart  | 2017-04-11  | 75    | 569           |
| mart  | 2017-04-09  | 68    | 569           |
| mart  | 2017-04-08  | 62    | 569           |
| neil  | 2017-05-10  | 12    | 581           |
| neil  | 2017-06-12  | 80    | 661           |

```

##### 

##### 查询前20%时间的订单信息

```sql
SELECT 
t1.name,
t1.orderdate,
t1.cost,
t1.group_num
from(
	SELECT 
	name,
	orderdate ,
	cost ,
	ntile(5)over(order by orderdate ) group_num
	from business
) t1
where t1.group_num=1;
jack	2017-01-01	10	1
tony	2017-01-02	15	1
tony	2017-01-04	29	1
```

##### 排名分析函数

```sql
    RANK()   dense_rank  row_number
100   1           1          1
100   1           1          2
99    3           2          3
98    4           3          4
97    5           4          5
96    6           5          6

RANK() 排序相同时会重复，总数不会变
DENSE_RANK() 排序相同时会重复，总数会减少
ROW_NUMBER() 会根据顺序计算

SELECT 
name ,
subject ,
score ,
rank() over(PARTITION  by subject ORDER  by score desc)  rk,
dense_rank()over(PARTITION  by subject ORDER  by score desc) drk,
row_number()over(PARTITION  by subject ORDER  by score desc) rrk
from score; 

孙悟空	数学	95	1	1	1
宋宋	数学	86	2	2	2
婷婷	数学	85	3	3	3
大海	数学	56	4	4	4
宋宋	英语	84	1	1	1
大海	英语	84	1	1	2
婷婷	英语	78	3	2	3
孙悟空	英语	68	4	3	4
大海	语文	94	1	1	1
孙悟空	语文	87	2	2	2
婷婷	语文	65	3	3	3
宋宋	语文	64	4	4	4
```

```sql
一.建表语句  
   partitioned by 表示创建的表是分区表
   clustered by  表是创建的表是分桶表 一般结合 Into num buckets(分成几个桶) 使用
二.排序
   order by 
   distribute by sort by  这叫以什么分区 区内以什么排序
   cluster by  分区排序(distribute by sort by 字段相同时可以简写成cluster by) 只能升序排
三.窗口函数
   partition by order by 窗口内分区排序
   distribute by sort by 
   
```

```sql
企业里面 ：用的是mr的引擎   orc+lzo
         用的是spark殷勤  parquet+snappy
```

