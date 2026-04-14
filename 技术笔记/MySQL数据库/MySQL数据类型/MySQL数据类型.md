# 数据类型

## 数据类型分类

- 数值类型

| 数据类型                      | 说明                                |
| ------------------------- | --------------------------------- |
| bit(M)                    | 位类型。M指定位数，默认值1，范围1-64             |
| tinyint [unsigned]        | 带符号的范围-128~127，无符号范围0~255，默认有符号   |
| bool                      | 使用0和1表示真和假                        |
| smallint [unsigned]       | 带符号是-2^15 ~ 2^15-1，无符号是2^16-1     |
| int [unsigned]            | 带符号是-2^31 ~ 2^31-1，无符号是2^32-1     |
| bigint [unsigned]         | 带符号是-2^63 ~ 2^63-1，无符号是2^64-1<br> |
| float (M, D) [unsigned]   | M指定显示长度，D指定小数数位，占用4字节<br>         |
| double (M, D) [unsigned]  | 比float精度更大的小数，占用8字节               |
| decimal (M, D) [unsigned] | M指定长度，D表示小数数位                     |

- 文本、二进制类型

- 时间日期

- string类型

## 数值类型


| 类型        | 字节  | 最小值                  | 最大值                  |
| --------- | --- | -------------------- | -------------------- |
|           |     | （带符号/无符号）            | （带符号/无符号）            |
| tinyint   | 1   | -128                 | 127                  |
|           |     | 0                    | 255                  |
| smallint  | 2   | -32768               | 32767                |
|           |     | 0                    | 65535                |
| mediumint | 3   | -8388608             | 8388607              |
|           |     | 0                    | 16777215             |
| int       | 5   | -2147483648          | 2147483647           |
|           |     | 0                    | 4294967295           |
| bigint    | 8   | -9223372036854775808 | 9223372036854775807  |
|           |     | 0                    | 18446744073709551615 |

### bit 类型

- 语法
```
bit[(M)] : 位字段类型。

M：表示每个值得位数，范围从1到64
如果M被忽略，默认为1
```

使用注意事项：
- bit字段在显示时，是按照ASCII码对应的值显示的
```mysql
mysql> insert into N1 values(65, 65);
mysql> select * from N1;
+------+------+
| id | a      |
+------+------+
| 10 |        |
| 65 | A      |
+------+------+
```
- 如果只存放0或1，表示真假，可以定义bit(1)。可以节省空间

### 小数类型

#### float

- 语法
```
float[(M, D)] [unsigned] : M指定显示长度，D指定小数位数，占用4个字节
```

- float(4, 2)表示的范围是 -99.99 ~ 99.99，MySQL 在保存值时会进行四舍五入
- 如果定义 float(4, 2) unsigned 无符号类型，范围是 0 ~ 99.99

#### decimal

- 语法：
```
decimal(M, D) [unsigned] : M指定长度，D表述小数点的位数
```

- decimal(5,2)表示的范围是-999.99~999.99
- decimal(5,2) unsigned 表示的范围 0~999.99

decimal和float很像，但是有区别：
- floa和decimal表示的精度不一样，float 表示的精度大约是7位
- decimal 整数最大位数m为65。支持小数最大位数d是30。如果d被省略，默认为0.如果m被省略，默认是10
- 如果希望小数的精度高，推荐使用 decimal

## 字符串类型

### char

- 语法：
```
char(L) : 固定长度字符串，L是可以存储的长度，单位为字符，最大长度值可以为255
```

- char(2) 表示可以存放两个字符，可以是字母或汉字

### varchar

- 语法：
```
varchar(L) : 可动态改变长度，L表述字符长度，最大长度65535个字节
```

varchar存储的真实长度：
- 由于是动态管理内存，需要用到 1 ~ 3 字节来记录数据大小，有效字节为 65532
- unf8：存放长度为 65532 / 3 = 21844 （一个字符占3个字节）
- gbk：存放长度为 65532 / 2 = 32766 （一个字符占2个字节）

### char 和 varchar


| 实际存储  | char(4) | varchar(4) | char占用字节   | varchar占用字节     |
| ----- | ------- | ---------- | ---------- | --------------- |
| abcd  | abcd    | abcd       | 4 * 3 = 12 | 4 * 3  + 1 = 13 |
| A     | A       | A          | 4 * 3 = 12 | 1 * 3 + 1 = 4   |
| abcde | error   | error      | 过长         | 过长              |

如何选择：
- 如果数据确定长度都一样，就使用定长(char)
- 如果数据长度有变化,就使用变长(varchar)
- 定长的磁盘空间比较浪费，但是效率高
- 变长的磁盘空间比较节省，但是效率低
- 定长的意义是，直接开辟好对应的空间
- 变长的意义是，在不超过自定义范围的情况下，用多少，开辟多少

### 日期和时间类型

- `date`：日期格式 `'XXXX-YY-ZZ'` 占用三个字节
- `datetime`：时间日期格式 `XXXX-YY-ZZ HH:II:SS` 表示范围从 `1000` 到 `9999`，占用八字节
- `timestamp`：时间戳，格式与 `datetime` 一致，占用四字节，日期随表的更改自动更新

```mysql
# 创建表
mysql> use test;
Database changed
mysql> create table t1(
    -> d1 date,
    -> d2 datetime,
    -> d3 timestamp
    -> );
Query OK, 0 rows affected (0.00 sec)

# 显示表
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| t1             |
+----------------+
1 row in set (0.00 sec)

# 插入数据
mysql> insert into t1 (d1, d2) values ('2000-01-01', '2020-01-01 14:23:00');
Query OK, 1 row affected (0.00 sec)

# 查看数据
mysql> select * from t1;
+------------+---------------------+---------------------+
| d1         | d2                  | d3                  |
+------------+---------------------+---------------------+
| 2000-01-01 | 2020-01-01 14:23:00 | 2026-04-13 19:48:41 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)

# 更新数据
mysql> update t1 set d1='2011-01-01';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# 时间戳更新
mysql> select * from t1;
+------------+---------------------+---------------------+
| d1         | d2                  | d3                  |
+------------+---------------------+---------------------+
| 2011-01-01 | 2020-01-01 14:23:00 | 2026-04-13 19:51:11 |
+------------+---------------------+---------------------+
1 row in set (0.00 sec)
```

### enum和set类型

#### enum

- 语法
```
对象 enum('选项1', '选项2'...);
```

- 可以通过**选项的值**插入，也可以通过对应的**数字编号**进行插入（选项1 -> 1，选项2 -> 2）
- **只能选择插入一个选项**，不能插入多个或不存在的值

#### set

- 语法
```
对象 set('选项1', '选项2'...);
```

- 通过 **选项的值 和 位图** 插入，不能通过数字编号进行插入
- 位图：由二进制的方式
	- 如果插入 1 这个数字，位图为 0001 （末尾的 1 = 选项1）
	- 如果插入 3 这个数字，位图为 0011 （末尾的 11 = 选项1 选项2）
- 可插入多个选项

#### 案例

```mysql
mysql> create table votes(
    -> username varchar(30),
    -> gender enum('男','女'),
    -> bobby set('C/C++','Java','Python','C#'));
Query OK, 0 rows affected (0.01 sec)

mysql> desc votes;
+----------+-----------------------------------+------+-----+---------+-------+
| Field    | Type                              | Null | Key | Default | Extra |
+----------+-----------------------------------+------+-----+---------+-------+
| username | varchar(30)                       | YES  |     | NULL    |       |
| gender   | enum('男','女')                   | YES  |     | NULL    |       |
| bobby    | set('C/C++','Java','Python','C#') | YES  |     | NULL    |       |
+----------+-----------------------------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

#### 插入数据

- 直接插入数据
```mysql
mysql> insert into votes values('myself','男','C/C++');
Query OK, 1 row affected (0.00 sec)

mysql> insert into votes values('peter','男','C/C++,Python');
Query OK, 1 row affected (0.01 sec)

mysql> insert into votes values('mitte','女','Java,Python');
Query OK, 1 row affected (0.00 sec)

mysql> select * from votes;
+----------+--------+--------------+
| username | gender | bobby        |
+----------+--------+--------------+
| myself   | 男     | C/C++        |
| peter    | 男     | C/C++,Python |
| mitte    | 女     | Java,Python  |
+----------+--------+--------------+
3 rows in set (0.00 sec)
```

- 通过数字和位图插入数据
```mysql
smysql> insert into votes values('anna', 2, 1);
Query OK, 1 row affected (0.01 sec)

mysql> insert into votes values('mia', 2, 2);
Query OK, 1 row affected (0.00 sec)

mysql> insert into votes values('david', 1, 3);
Query OK, 1 row affected (0.01 sec)

mysql> insert into votes values('max', 1, 4);
Query OK, 1 row affected (0.00 sec)

mysql> insert into votes values('list', 1, 8);
Query OK, 1 row affected (0.00 sec)

mysql> insert into votes values('string', 1, 7);
Query OK, 1 row affected (0.00 sec)

mysql> select * from votes;
+----------+--------+-------------------+
| username | gender | bobby             |
+----------+--------+-------------------+
| myself   | 男     | C/C++             |
| peter    | 男     | C/C++,Python      |
| mitte    | 女     | Java,Python       |
| anna     | 女     | C/C++             |
| mia      | 女     | Java              |
| david    | 男     | C/C++,Java        |
| max      | 男     | Python            |
| list     | 男     | C#                |
| string   | 男     | C/C++,Java,Python |
+----------+--------+-------------------+
7 rows in set (0.00 sec)
```

#### 查询数据

- 语法：查对应数据
```
select * from [表] where [对象] = [数据]
```

```mysql
mysql> select * from votes where bobby = 'C/C++';
+----------+--------+-------+
| username | gender | bobby |
+----------+--------+-------+
| myself   | 男     | C/C++ |
| anna     | 女     | C/C++ |
+----------+--------+-------+
2 rows in set (0.00 sec)
```

- 语法：查集合内的数据
```
select * from [表] where find_in_set('数据', '对象');
```

```
select * from [表] where find_in_set('数据', '对象') and find_in_set('另外的数据', '对象');
```

- 查询喜爱 `C/C++` 的人
```mysql
mysql> select * from votes where find_in_set ('C/C++', bobby);
+----------+--------+-------------------+
| username | gender | bobby             |
+----------+--------+-------------------+
| myself   | 男     | C/C++             |
| peter    | 男     | C/C++,Python      |
| anna     | 女     | C/C++             |
| david    | 男     | C/C++,Java        |
| string   | 男     | C/C++,Java,Python |
+----------+--------+-------------------+
5 rows in set (0.00 sec)
```

- 查询喜爱 `C/C++、Java` 的人
```
mysql> select * from votes where find_in_set ('C/C++', bobby) and find_in_set ('Java', bobby);

+----------+--------+-------------------+
| username | gender | bobby             |
+----------+--------+-------------------+
| david    | 男     | C/C++,Java        |
| string   | 男     | C/C++,Java,Python |
+----------+--------+-------------------+
2 rows in set (0.00 sec)
```