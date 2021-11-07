## CH简介

俄罗斯Yandex公司，列式储存数据库，C++编写，用于OLAP，SQL查询

特点：

1. 列式储存
2. DBMS功能 覆盖大多数SQL用法
3. 多样化引擎 合并树、日志、接口和其他四大类  20 多种引擎
4. 高吞吐写入  LSM Tree 的结构 写入时顺序写入 定期在后台 Compaction时merge sort 后顺序写回磁盘
5. 数据分区 线程并行 单条 Query 就能利用整机所有 CPU

## 特性说明

非常吃CPU主要瓶颈，因此适合处理处理后的数据，即宽表

适合单表查询，不适合做join

## 安装

1. 关闭防火墙
2. 取消打开文件数限制

在/etc/security/limits.conf 文件下添加如下信息

```
* soft nofile 65536 
* hard nofile 65536 
* soft nproc 131072 
* hard nproc 131072
# *表示用户组 所有   可以用user1@usergp
# soft当前hard上限 soft小于等于hard file是文件数 proc是进程数

# 重新登录生效
```

注意20.5支持final，20.6.3支持explain执行计划，20.8新引擎支持实时同步MySQL

```linux
# 修改配置文件 将注释去掉
sudo vim /etc/clickhouse-server/config.xml

# 需要将下面注释去掉
<listen_host>::</listen_host>
```

注意几个默认路径

数据文件：<path>/var/lib/clickhouse/</path>

日志文件：<log>/var/log/clickhouse-server/clickhouse-server.log</log>

    # 文件对应目录
    bin/    ===>  /usr/bin/ 
    conf/   ===>  /etc/clickhouse-server/
    lib/    ===>  /var/lib/clickhouse 
    log/    ===>  /var/log/clickhouse-server



```linux
# 启动服务
sudo systemctl start clickhouse-server
# 连接客户端
clickhouse-client -m
```

## 数据类型

### 整型

| ClickHouse | Java  | 占用空间bit |
| ---------- | ----- | ----------- |
| Int8       | byte  | 8           |
| Int16      | short | 16          |
| Int32      | int   | 32          |
| Int64      | long  | 64          |

### 无符号整型

以上数据类型还有无符号类型，是在前面加U，UInt8，UInt16，UInt32，UInt64

浮点型

| ClickHouse | Java   | 占用空间bit |
| ---------- | ------ | ----------- |
| Float32    | Float  | 32          |
| Float64    | Double | 64          |

浮点型存在精度丢失问题

### 布尔型

用UInt8表示，0位false，1为true

### Decimal 能够保证精度

Decimal32(s)  s表示小数点后的位数 有效位1-9 

Decimal64(s)  s表示小数点后的位数 有效位1-18

Decimal128(s)  s表示小数点后的位数 有效位1-38

其中有效位包含了小数位

### 字符串

String

FixedString(N) 固定长度，不够用添加空字符串，长度大于N返回错误信息

### 枚举类型

Enum8  String=Int8

Enum16  String=Int16

枚举类型相当于字符串与整型的对应关系，可以节省空间

### 时间类型

| 时间类型   | 格式                   | 例子                   |
| ---------- | ---------------------- | ---------------------- |
| Date       | 年-月-日               | 2020-10-10             |
| Datetime   | 年-月-日 时：分：秒    | 2020-10-10 10:10:10    |
| Datetime64 | 年-月-日 时:分:秒.亚秒 | 2020-10-10 10:10:10.66 |

### 数组

Array(T)，CH对多维数组支持有限

### Nullable

会对性能造成负面影响，一般针对数字用没有意义值代替 如-1，字符串用空串

### MergeTree

最强大的，特别说明 primary key是可以重复的

```sql
create table t_order_mt( 
id UInt32,
sku_id String,
total_amount Decimal(16,2), 
create_time Datetime
) engine =MergeTree
partition by toYYYYMMDD(create_time) 
primary key (id)
order by (id,sku_id);
```

其中：primary key 不是唯一性的，提供一级索引，但不是唯一约束，稀疏索引，默认分隔粒度8192

```
# 建表时默认添加
SETTINGS index_granularity = 8192
```

order by是根据partition by中进行排序的，因为用到了稀疏索引，因此需要设置排序的列

**注意**：主键必须是order by字段的前缀字段，顺序一致，不能跳过排序的前面字段，只看排序后面字段可能无序

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2,
    ...
    PROJECTION projection_name_1 (SELECT <COLUMN LIST EXPR> [GROUP BY] [ORDER BY]),
    PROJECTION projection_name_2 (SELECT <COLUMN LIST EXPR> [GROUP BY] [ORDER BY])
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr
    [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx' [, ...] ]
    [WHERE conditions]
    [GROUP BY key_expr [SET v1 = aggr_func(v1) [, v2 = aggr_func(v2) ...]] ] ]
[SETTINGS name=value, ...]
```

PARTITION BY分区，不写为一个分区，默认文件名为all，避免全表扫描，clickhouse分目录，存放在本地磁盘，由列文件+索引文件+表定义文件组成



文件存放位置

```
# docker下需进入service所在容器，用下面方式
docker exec -it id /bin/bash

# 所在目录
/var/lib/clickhouse
# 主要关注下面两个文件
drwxr-x---  5 clickhouse clickhouse 4096 Nov  7 11:42 data/
drwxr-x---  2 clickhouse clickhouse 4096 Nov  7 11:42 metadata/
```

库名以文件夹的形式储存，在data/下

进入表的目录

```linux
drwxr-x--- 2 clickhouse clickhouse 4096 Nov  7 11:43 20200601_1_1_0/
drwxr-x--- 2 clickhouse clickhouse 4096 Nov  7 11:43 20200602_2_2_0/
drwxr-x--- 2 clickhouse clickhouse 4096 Nov  7 11:42 detached/
-rw-r----- 1 clickhouse clickhouse    1 Nov  7 11:42 format_version.txt
```

进入分区内查看信息

```linux
 cd 20200601_1_1_0
 
-rw-r----- 1 clickhouse clickhouse  259 Nov  7 11:43 checksums.txt  # 校验信息
-rw-r----- 1 clickhouse clickhouse  118 Nov  7 11:43 columns.txt  # 列的信息
-rw-r----- 1 clickhouse clickhouse    1 Nov  7 11:43 count.txt  # 多少行
-rw-r----- 1 clickhouse clickhouse  189 Nov  7 11:43 data.bin   # 数据文件
-rw-r----- 1 clickhouse clickhouse  144 Nov  7 11:43 data.mrk3   # 标记文件 方便查找，在idx索引文件和bin数据文件起桥梁作用
-rw-r----- 1 clickhouse clickhouse   10 Nov  7 11:43 default_compression_codec.txt
-rw-r----- 1 clickhouse clickhouse    8 Nov  7 11:43 minmax_create_time.idx   # 分区键的最大最小值
-rw-r----- 1 clickhouse clickhouse    4 Nov  7 11:43 partition.dat  # 分区信息
-rw-r----- 1 clickhouse clickhouse    8 Nov  7 11:43 primary.idx  # 索引文件 稀疏索引
```

注意：任何一个批次写入，会写入一个临时分区，不会纳入已有分区，写入后某个时刻再执行合并操作，也可以手动设置

```linux
optimize table xxx final;
```

### 二级索引

```sql
create table t_order_mt2( 
id UInt32,
sku_id String,
total_amount Decimal(16,2), 
create_time  Datetime,
-- 创建索引 GRANULARITY 表示二级索引 是对索引的索引 类似跳表
INDEX a total_amount TYPE minmax GRANULARITY 5 
) engine =MergeTree
partition by toYYYYMMDD(create_time)
primary key (id) 
order by (id, sku_id);
```

查看是否有二级索引

```sql
show create table_name;

# 切换到对应分区目录下，查看是否有如下文件
-rw-r----- 1 clickhouse clickhouse   41 Nov  7 14:28 skp_idx_a.idx
-rw-r----- 1 clickhouse clickhouse   24 Nov  7 14:28 skp_idx_a.mrk3
```

### 数据TTL

TTL：Time To Live 数据的生命周期

列级别和表级别，到时间后的处理方式

### ReplacingMergeTree

只能保证最终一致性，因为去重是在合并进行的，无法预估合并时间，只能用order by的字段，而且只能分区内去重，不能保证没有重复数据

- 去重不跨分区
- 同一批插入或合并分区时 去重
- 保留版本字段最大
- 版本字段相同保留插入最后

P15

