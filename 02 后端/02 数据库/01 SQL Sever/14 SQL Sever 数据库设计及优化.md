# SQL Sever 数据库设计及优化

## 1. 数据库设计

### 1.1 表关系

> 表与表之间的关联关系，主要有如下三种情况

* 1 对 1
* 1 对 多，一般是由主、外健形成关系。
* 多 对 多，多对多关系的表，一般会形成一个中间表，例如：学生表与课程表，中间会形成分数表。

### 1.2 数据库范式

> 为了规范数据库设计，设定了三种范式。
>
> 注：数据库设计得根据具体系统要求进行分析，并不一定非要严格遵守三范式。有时可以牺牲某个范式设计，使得系统更易于修改和操作。其中的利弊需要根据实际情况进行权衡。

#### 1.2.1 第一范式 (1NF)

> 每一行数据，各字段 (列) 数据都不可再分割，保证数据具有原子性。

例如：

* 在有些系统中，“地址”列还可以细分为国家、省、市、区等列。

* 国外有的程序“姓名”列也需要拆分为“姓”和“名”等。

#### 1.2.2 第二范式 (2NF)

> 在满足第一范式的情况下，除了主键(引用其他表主键做外键也算)外的其他所有列，都和主键数据相关。
>
> 例如：学生表，添加一个颜色属性，这明显就是无关字段。

#### 1.2.3 第三范式 (3NF)

> 在满足第二范式的情况下，各列数据都和主键数据强相关，限制列 的冗余性。
>
> 即，如果一个表与另一个表有关联，那么引入另一个表的主键ID一个列作为外键即可，另一个表的其他列不用存在该表中。

### 1.3 设计工具推荐

* **PowerDesigner**

    > 可以创建表关联关系，并生成创建代码。

## 2. SQL 语句优化

1. 尽量避免隐式的数据类型转换

````Sql
-- 例: employee 表的 salary 类型是 float 类型
-- 坏
select emp_name from employee where salary>3000
-- 优化写法
select emp_name from employee where salary>3000.0
````

2. 尽量避免select *

> * 会增加解析的时间，
> * 另外会把不需要的数据也给查询出来
> * 用select *做了批量插入操作，如果后续新增了字段，之前写的代码可能会出问题。 

3. 谨慎使用模糊查询

> * 使用前 % 号，会使索引失效，引起全表扫描。

````sql
-- 不推荐
select * from employee where emp_name like '%paral%'

-- 推荐
select emp_name tel from employee where emp_name like 'paral%'
````

4. 应尽量避免在 where子句中对字段进行函数或者表达式操作

> * 对字段进行函数以及运算操作，会使索引失效，引起全表扫描。
> * 函数操作尽量对常量使用。

````sql
-- 坏
select id from t where substring(name,1,3)='abc'
select id from t where datediff(day,createdate,'2022-05-01')=0
select id from where num/2=100

-- 优化
select id from t where name like 'abc%'
select id from t where createdate>='2022-01-01' and createdate<='2022-05-01'
select id from t where num=100*2
````

5. 避免在 where 子句中对字段进行 null 值判断

> * 对字段进行 null 值判断，会使索引失效，引起全表扫描。
> * 在建表时，字段都应该添加Not Null 约束，然后设定默认值来替代 null。

````sql
-- 不推荐
select id from t where num is null

-- 推荐
select id from t where num=默认值 -- 前提是设定了 num 字段为not null,且默认值为0
````

5. 尽量避免在 where 子句中使用 or 来连接条件

> * 使用 or 来连接条件，会使索引失效，引起全表扫描。
> * 可以使用 UNION ALL 或者 UNION 来替代 or。

````sql
-- 不推荐
select id from t where num=10 or num=20

-- 推荐
select id from t where num=10
union all
select id from t where num=20
````

6. 避免 where 子句中使用变量参数。

> where 子句中使用变量参数，会使索引失效，引起全表扫描。
>
> 因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时，它必须在编译时进行选择。
>
> 然而，如果在编译时建立访问计划，变量的值还是未知的， 因而无法作为索引选择的输入项。

````sql
-- 坏
select id from t where num=@num
-- 优化
select id from t with(index(索引名)) where num=@num -- 强制使用索引
````

7. 优先使用UNION ALL，避免使用UNION

> UNION 有个去重操作，影响性能，能使用UNION ALL的话则优先考虑。

8. 很多时候用 exists 代替 in 是一个好的选择

> * in() 括号中，()中适合存放少量数据
> * exists() 括号中为真即可，也适合进行表关联查询。

9. 当业务涉及非常多的表操作时，可以使用存储过程。

> * 减少网络IO、减少网络阻塞。
> * 存储过程具有预编译效果，通过缓存计划并在重复执行时不用来降低编译开销，执行速度加快。
> * 易于修改和维护。

## 3. 表优化—表分区

### 3.1 概念

> 默认情况下，表所有的数据都保存在主文件组的主文件中。
>
> 数据查询都在一个文件中查找，随着数据量的不断增大，文件会越来越大，查询速度也会逐渐降低。
>
> 对表进行分区，即将表中数据，按一定的规则分别存储到不同的数据库文件中。
>
> 这样，查询前可根据规则去保存有该数据文件中查询即可，能大大提高对大数据量表的查询速度。

### 3.2 分区步骤

> * 为数据库创建多个文件组、以及对应的数据库文件
>
> * 创建分区函数，指定分区所依据的列字段的数据类型、分区数、以及数据边界值。
>
> * 创建分区策略，指定分区采用的分区函数，以及对应的文件组。
>
> * 创建分区表。
>
> * 如果是已经存在的表，则需要进行以下操作：
>
>     * 删除原有表的主键约束、以及索引。
>
>     * 按照分区策略重建分区列的索引。
>
>     * 重建分区列主键约束。

````sql
-- 分区函数
-- LEFT: 表示分区函数按照分区键的左边界进行分区。
-- RIGHT: 表示分区函数按照分区键的右边界进行分区
CREATE PARTITION FUNCTION <partition_function_name>(<parameter_data_type>)
AS RANGE LEFT | RIGHT
FOR VALUES (<boundary_value>[, ...n ]);
````

````sql
-- 分区策略
CREATE PARTITION SCHEME <partition_scheme_name>
AS PARTITION <partition_function_name>TO (<filegroup_name> [,...n ] )
````

````sql
-- 创建分区存储的表
CREATE TABLE <table_name>
(
    <column1> <data_type> [ , ...n ]
)
ON <partition_scheme_name> (<column_name>)
````

### 3.3 分区管理

#### 3.3.1 切换分区

> 切换分区（SWITCH PARTITION）：可以通过切换分区操作将数据从一个分区移动到另一个分区。
>
> 这对于数据加载、归档和删除等操作非常有用。例如，可以将旧的数据归档到历史分区中。

````sql
ALTER TABLE <table_name>
SWITCH PARTITION <source_partition_number> TO <target_partition_number>;
````

#### 3.3.2 合并分区

> 合并分区（MERGE PARTITION）：如果某个分区中的数据变得过少或不再使用，可以通过合并分区操作将该分区与相邻的分区合并。这可以减少分区的数量，并降低存储成本。

````sql
ALTER TABLE <table_name>
MERGE PARTITION <source_partition_number> TO <target_partition_number>;
````

#### 3.3.3 分割分区

> 分割分区（SPLIT PARTITION）：如果某个分区中的数据增多，可以通过分割分区操作将该分区拆分为两个或多个分区。这可以提供更好的性能和管理能力。

````sql
ALTER TABLE <table_name>
SPLIT PARTITION <source_partition_number> AT (<split_value>);
````

#### 3.3.4 增加分区

> 增加分区（ADD PARTITION）：可以通过增加分区操作为分区表添加新的分区。这对于扩展数据存储和管理非常有用。

```sql
ALTER TABLE <table_name>
ADD (PARTITION <partition_name> VALUES <boundary_value>);
```

### 3.4 代码案例

````sql
-- 假设有张大数据表：Account，现以它的 ID（bigint类型） 字段为分区列进行分区。

-- 1.创建文件组
ALTER DATABASE [BigDataBase] ADD FILEGROUP FG1
GO
ALTER DATABASE [BigDataBase] ADD FILEGROUP FG2
GO
ALTER DATABASE [BigDataBase] ADD FILEGROUP FG3
GO
ALTER DATABASE [BigDataBase] ADD FILEGROUP FG4
GO
ALTER DATABASE [BigDataBase] ADD FILEGROUP FG5 -- 兜底
GO
-- 2.为各文件组，创建数据文件
ALTER DATABASE [BigDataBase]
ADD FILE
(
	NAME=Account1,
	FILENAME=N'C:\TestDATA\Account1.ndf',
	SIZE=5mb,
	MAXSIZE=500mb,
	FILEGROWTH=50mb
)
TO FILEGROUP FG1
GO
ALTER DATABASE [BigDataBase]
ADD FILE
(
	NAME=Account2,
	FILENAME=N'C:\TestDATA\Account2.ndf',
	SIZE=5mb,
	MAXSIZE=500mb,
	FILEGROWTH=50mb
)
TO FILEGROUP FG2
GO
ALTER DATABASE [BigDataBase]
ADD FILE
(
	NAME=Account3,
	FILENAME=N'C:\TestDATA\Account3.ndf',
	SIZE=5mb,
	MAXSIZE=500mb,
	FILEGROWTH=50mb
)
TO FILEGROUP FG3
GO
ALTER DATABASE [BigDataBase]
ADD FILE
(
	NAME=Account4,
	FILENAME=N'C:\TestDATA\Account4.ndf',
	SIZE=5mb,
	MAXSIZE=500mb,
	FILEGROWTH=50mb
)
TO FILEGROUP FG4
GO
ALTER DATABASE [BigDataBase]
ADD FILE
(
	NAME=Account5,
	FILENAME=N'C:\TestDATA\Account5.ndf',
	SIZE=5mb,
	MAXSIZE=500mb,
	FILEGROWTH=50mb
)
TO FILEGROUP FG5
GO

-- 3. 将其中一个文件组设置为默认文件组
ALTER DATABASE [BigDataBase]
MODIFY FILEGROUP FG5 DEFAULT
GO

-- 4. 创建分区函数
/*
*分区规则为：
*分区表中的Id范围为1-2000000的数据被分到FG1中
*分区表中的Id范围为2000001-4000000数据被分到FG2中
*分区表中的Id范围为4000001-6000000的数据被分到FG3中、
*分区表中的Id范围为4000001-8000000的数据被分到FG4中
*分区表中的Id范围为8000001 以后的数据被分到FG5中
*/
-- 分区函数名ACCOUNT_FUC，分区列数据类型BIGINT
CREATE PARTITION FUNCTION ACCOUNT_FUC(BIGINT) 
AS RANGE LEFT FOR VALUES(2000000,4000000,6000000,8000000) -- 边界值，8000001之后边界不确定不用写。
GO

-- 5. 创建分区方案
CREATE PARTITION SCHEME ACCOUNT_SCHEME -- 分区方案名称 ACCOUNT_SCHEME
AS 
PARTITION ACCOUNT_FUC TO(FG1,FG2,FG3,FG4,FG5) -- 采用分区函数ACCOUNT_FUC，设置匹配的文件组
GO

-- 6. 解除原表的主键、索引、以及索引
-- 解除主键
ALTER TABLE dbo.Account
DROP CONSTRAINT [PK__Account__3214EC071F785A8D]
GO
-- 查看索引情况
EXEC sys.sp_helpindex @objname = N'Account' -- nvarchar(776)
-- 删除索引
DROP INDEX i_debtor ON dbo.Account

-- 7. 按创建的分区策略来重建索引。（分区后，其他列的索引正常创建即可）
CREATE CLUSTERED INDEX IDX_ACCOUNT_ID ON dbo.Account(ID) ON ACCOUNT_SCHEME(ID)
GO

-- 8. 重新设置约束
ALTER TABLE dbo.Account
ADD CONSTRAINT PK_ACCOUNT_ID PRIMARY KEY(ID)
GO

-- 9. 分区完成，查看该表分区情况
SELECT $partition.ACCOUNT_FUC(ID) as partition_num,count(*) as record_num
from dbo.Account
GROUP BY $partition.ACCOUNT_FUC(ID)
ORDER BY $partition.ACCOUNT_FUC(ID)
````

## 4. 其他优化方案

### 4.1 临时表优化

> 当需要查询某个大表中部分数据，后续用这些数据进行逻辑操作时，可以先将这些数据查出存放到临时表，后续操作对临时表操作，避免大表的重复查询。

### 4.2 表水平拆分

> 某些表数据量比较大时，可以同时建立结构相同的多个表用于存储。
>
> 使用相同前缀不同的后缀来进行表命名，按时间或者其他条件进行分表存储。
>
> 将这些表通过UNION ALL 创建视图，以供后续进行统一查询。

````sql
-- 例
-- 创建结构相同的数据表，这里以日期划分存储，每月的数据备份迁移存储到对应的的表。
CREATE TABLE Logs
(
	LogId INT PRIMARY KEY,
	LogType NVARCHAR(100),--日志类型
	Operator NVARCHAR(100),--操作者
	ItemId INT,--操作项主键
	[Description] NVARCHAR(2000),--操作描述
	OperateDate DATETIME,--操作日期
	OperateIP NVARCHAR(50)--操作IP
)
CREATE TABLE Logs_202311
(
	LogId INT PRIMARY KEY,
	LogType NVARCHAR(100),--日志类型
	Operator NVARCHAR(100),--操作者
	ItemId INT,--操作项主键
	[Description] NVARCHAR(2000),--操作描述
	OperateDate DATETIME,--操作日期
	OperateIP NVARCHAR(50)--操作IP
)
GO
CREATE TABLE Logs_202312
(
	LogId INT PRIMARY KEY,
	LogType NVARCHAR(100),--日志类型
	Operator NVARCHAR(100),--操作者
	ItemId INT,--操作项主键
	[Description] NVARCHAR(2000),--操作描述
	OperateDate DATETIME,--操作日期
	OperateIP NVARCHAR(50)--操作IP
)
GO
-- 创建视图，关联起所有的日志子表，查询时直接操作此视图
CREATE VIEW Logs
AS
	SELECT * FROM dbo.Logs UNION ALL
	SELECT * FROM dbo.Logs_202311 UNION ALL
	SELECT * FROM dbo.Logs_202312
GO

-- 创建存储过程，每月月初按该表结构，自动创建一张新表，并将Logs主表中上一个月的数据迁移到该表中。
-- 略
````

### 4.3 表垂直拆分

> 有时，一个有多个列字段的表，有时候可以将这些列拆分成两个表进行存储。再在两个表之间建立一对一的关联关系。
>
> 这样，可以将经常查询但不经常修改的列划分在一个表，并建立索引提高查询效率；将经常修改的表划分在另外一个列，方便修改。

### 4.4 垂直拆库

> 将大业务系统按功能模块划分，不同模块的数据，创建不同的数据库保存。
>
> 这样，可以减轻单台数据库的访问压力，系统模块也更加容易扩展。

### 4.5 主从备份（发布订阅）

> 主从备份，该模式也称读写分离、发布订阅、建立集群。
>
> * 设立两个数据库服务器，两个数据库数据保持一致。
>
> * 将其中一个数据库设为主库，主要负责进行增删改的操作。
>
> * 将另外一个数据库设置为只读的辅助库，主要负责读取操作。
> * 在主库服务器崩了时，系统可以自动切换到辅库，将辅库转为主库使用。
>
> 这样，读写操作分别请求了不同服务器，减少了单台服务器的压力，提高了系统的可用性。
>
> 缺点：主库辅库数据库同步可能需要时间，查询会有延迟。