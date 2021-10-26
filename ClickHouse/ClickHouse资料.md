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

