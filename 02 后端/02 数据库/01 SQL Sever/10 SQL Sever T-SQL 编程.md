# SQL Sever T-SQL 编程

## 1. 变量定义

### 1.1 命名规则

> * 变量由 @ 符号开头。
> * 首字符后可以包含 字母 数字 汉字 _ @ #。
> * 不能是系统保留字（关键字）。

### 1.2 全局变量

> 全局变量一般由系统定义，在 SQL Sever 全局都可用。
>
> @@ 开头的并不一定是系统变量，有些是系统函数，二者作用都是返回SQL Sever系统相关数据。
>
> 常见的全局变量或系统函数如下：

````sql
@@ERROR -- 返回执行的上一个T-SQL语句的错误号,0 代表没有错误，非0代表有错误

@@ROWCOUNT -- 返回上一条语句影响的数据行的行数

@@IDENTITY -- 返回前面插入语句插入到表的IDENTITY列的最后一个值

@@LANGUAGE -- 返回当前所用语言的名称
@@MAX_CONNECTIONS -- 返回Sql Server实例允许同时进行的最大用户连接数
@@SERVERNAME -- 返回运行Sql Server的本地服务器的名称
@@SERVICENAME -- 返回Sql Server 正在其下运行的注册表项的名称
@@TIMETICKS -- 返回每个时钟周期的微秒数
@@TRANCOUNT -- 返回当前连接的活动事务数
@@VERSION -- 返回当前安装的日期、版本和处理器类型
````

### 1.3 局部变量

> 局部变量以 1 个 @ 符号开头，仅当前连接有用，连接断开即被清除。

````sql
-- 定义并初始化变量
DECLARE @name NVARCHAR(50)=''

-- 变量赋值方式 1
SET @name = '张三'

-- 变量赋值方式 2
SELECT @name = name FROM 表 WHERE 筛选条件
````

## 2. 运算

> 正常的加减乘除、大小判断和其他编程没什么区别。

### 2.1 不等于

> 在SQL Sever中，不等于号写 `<>` 或者 `!=` 都可以。

### 2.2 字符串连接

> SQL Sever 中数字 + 字符串不会拼接成字符串，而是会将字符串转换成数字进行相加。

````sql
select '10'+'10'; -- 输出结果：1010

select 10+'10'; -- 输出结果：20

select 10+'10a'; -- 报错，还是因为没有隐式转换
````

## 3. 流程控制

### 3.1 if 结构

> * SQL Sever 中用 BEGIN ... END 替代{ }符号，来表示代码区块。
> * 执行语句只有一条时，BEGIN ... END 可以省略。
> * IF 后的 () 可以省略，但是如果条件内包含select语句 ，()不能省略。

````SQL
IF (条件)
BEGIN
	语句块
End
ELSE IF (条件2)
BEGIN
	语句块
End
````

### 3.2 case 结构

> case 不能像 IF 结构一样直接使用，得配合 SELECT 语句。

````sql
-- 方式一: 一般用于等值判断
case 字段
	when 值1 then 结果
	when 值2 then 结果
	else 结果
end

-- 方式二：一般用于区间范围比较
case
	when 包含字段的条件判断 then 结果
	when 包含字段的条件判断 then 结果
	else 结果
end
````

````sql
-- 例
DECLARE @score INT = 77

SELECT 
CASE 
	WHEN @score >= 0 AND @score < 60 THEN '差'
	WHEN @score >= 60 AND @score < 80  THEN '良'
	WHEN @score >= 80 AND @score < 100 THEN '优'
	ELSE '分数不在区间'
END
````

### 3.3 循环结构

> * while后面的 () 可以省略，但是如果循环条件内含有select语句，()不能省略
> * 可以使用 break；和 continue；关键字控制跳出 循环

````sql
while (条件)
begin
	-- 语句块
end
````

### 3.4 等待

> `waitfor delay `用于延迟执行后方语句

````sql
print '1';
waitfor delay '00:00:10'; -- 等10秒执行
print '2';
````

## 4. 函数

### 4.1 系统函数

> 以下列举常见或者有可能用上的函数，详细的其他系统函数可进SQL Sever数据库中查看。

#### 4.1.1 聚合函数

| 函数    | 作用               |
| ------- | ------------------ |
| Avg()   | 求组中平均值       |
| Count() | 求组中数据行数     |
| Max()   | 求组中最大值       |
| Min()   | 求组中最小值       |
| Sum()   | 求组中所有数据总和 |

#### 4.1.2 配置函数

| 函数              | 作用                                           |
| ----------------- | ---------------------------------------------- |
| @@Language        | 获取当前SQL Sever使用语言名称，                |
| @@Max_Connections | 获取当前系统允许的最大连接数，默认32767        |
| @@Max_Precision   | 返回设置的decimal和numeric数据精度级别，默认38 |
| @@Servername      | 返回服务器SQL Sever名称                        |
| @@Servicename     | 返回服务器SQL Sever注册的服务名称              |
| @@Spid            | 返回当前用户进程的服务器进程id                 |
| @@Version         | 返回服务器版本信息                             |

#### 4.1.3 日期和时间函数

> 下方的“部分“，传对应年月日时分秒的英文即可。

| 函数                              | 返回类型 | 作用                                                 |
| --------------------------------- | -------- | ---------------------------------------------------- |
| Current_Timestamp                 | datetime | 返回当前时间，格式 YYYY-MM-DD HH:mm:ss.xxx           |
| Getdate()                         | datetime | 返回当前时间，格式 YYYY-MM-DD HH:mm:ss.xxx           |
| Year(时间)                        | int      | 获取传入时间的年份                                   |
| Month(时间)                       | int      | 获取传入时间的月份                                   |
| Day(时间)                         | int      | 获取传入时间的日期（天）                             |
| Datename(部分,时间)               | varchar  | 获取传入时间的指定部分，年\月\日\时\分\秒            |
| Datepart(部分,时间)               | int      | 获取传入时间的指定部分，年\月\日\时\分\秒            |
| Dateadd(部分,数值,时间)           | datetime | 在传入的时间基础上，在指定部分加上指定数值。         |
| Datediff(部分,开始日期，结束日期) | datetime | 获取两个时间间隔，并且输出指定间隔部分               |
| Isdate('字符串')                  | int      | 判断当前输入的格式是否是时间格式，1表示是，0表示不是 |

#### 4.1.4 数学函数

| 函数              | 作用                                 |
| ----------------- | ------------------------------------ |
| Abs()             | 求绝对值                             |
| Ceiling()         | 向上取整                             |
| Floor()           | 向下取整                             |
| Power(数值，幂值) | 求次方                               |
| Rand()            | 返回0-1之间的随机小数，返回float类型 |
| Round(数值，精度) | 四舍五入到指定精度                   |
| Sqrt()            | 开平方根                             |
| Square()          | 求平方                               |

#### 4.1.5 字符串函数

> 注：SQL Sever 操作字符串的下标是从1开始，不是从0开始

| 函数                                 | 作用                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| Char(数值)                           | 将ASCII整数转换为char字符                                    |
| **Str**(float,有效数位,小数数位)     | 将小数转换成字符串                                           |
| **Reverse**(str)                     | 字符串顺序反转                                               |
| **Len**(str)                         | 返回字符串的字符数，长度                                     |
| Lower(str)                           | 将字符串全部转换为小写                                       |
| Upper(str)                           | 将字符串全部转换为大写                                       |
| Ltrim(str)                           | 删除字符串前面的空格                                         |
| Rtrim(str)                           | 删除字符串后面的空格                                         |
| **Trim**()                           | 删除字符串前后的空格                                         |
| **Replace**(str,sonStr,newStr)       | 将字符串中的指定字符串，替换成新字符串。                     |
| **Stuff**(str,start,length,newStr)   | 将字符串中指定位置开始指定长度的字符串替换成新的字符串。     |
| **Left**(str,length)                 | 从左截取指定长度的字符串                                     |
| **Right**(str,length)                | 从右截取指定长度的字符串                                     |
| **Substring**(str,start,length)      | 从指定位置截取指定长度的字符串                               |
| Concat(str1,str2,...)                | 将N个字符串拼接                                              |
| Concat_Ws(拼接符,str1,str2,...)      | 将N个字符串用拼接符拼接                                      |
| **String_Agg**(字段,拼接符)          | 可在查询语句中，将查询的某列字段的所有数据用拼符拼接成一条数据。 |
| Replicate(str,n)                     | 将字符串复制 n 次                                            |
| **CharIndex**(sonStr,str,startIndex) | 从指定位置开始搜索，返回子字符串在字符串中第一次出现的下标位置 |
| **String_Split**(str,分隔符)         | 将字符串按分割符进行分割，并用一个单列表返回                 |

#### 4.1.6 元数据函数

| 函数      | 作用                     |
| --------- | ------------------------ |
| Db_Name() | 获取当前操作的数据库名称 |
|           |                          |

#### 4.1.7 其他函数

| 函数                                     | 作用                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| **Convert**(N'数据类型',数据,[日期格式]) | 将数据转换成指定格式，如果是日期则可以添加第三个参数来定义格式 |
| **ISNULL**(数据,新值)                    | 将为null的数据替换成新值。                                   |
| isnumeric(数据)                          | 判断是否是有效数字数据                                       |
| Cast(数据,N'数据类型')                   | 将数据转换成另一种数据类型（此方式不常用）                   |
| @@Error                                  | 上一个SQL 语句返回的错误号                                   |
| @@Identity                               | 返回上一条语句插入的唯一标识。                               |
| **@@Rowcount**                           | 返回上一条数据影响的行数                                     |
| @@Trancount                              | 返回当前连接的事务数                                         |
| Host_Id()                                | 返回连接的电脑ID                                             |
| **Host_Name**()                          | 返回连接的电脑名称                                           |
| **App_Name**()                           | 返回当前会话应用程序名称（例如SSMS、QQ等）                   |
| **SYSTEM_USER**                          | 返回当前系统登录名（数据库账号）                             |
| Newid()                                  | 创建uniqueidentify类型的唯一标识。                           |
| **EventData()**                          | xml格式，获取事件操作记录。                                  |



### 4.2 自定义函数

> 函数按返回值类型可分为标量值函数和表值函数
>
> 标量值函数：返回值是一个单个的值
>
> 表值函数：返回值是表类型数据

````SQL
-- 创建标量值函数
CREATE FUNCTION 函数名(参数1 参数类型...) RETURNS 返回值类型
BEGIN
	-- 处理逻辑
    RETURN 返回值;
END
GO
````

```sql
-- 创建表值函数
CREATE FUNCTION 函数名(参数1 参数类型...) RETURNS TABLE
AS
	-- 处理逻辑
    RETURN (SELECT ... 语句);
GO
```

## 5. 批量操作

### 5.1 批量导入新表

> 从表 1 中查询出一系列数据，创建字段结构与查询字段相同的表 2，并且将查询出的数据插入到表 1。
>
> 要求：表 1 必须存在，**表 2 必须不存在** 

````sql
select 字段1,字段2,字段3,... into #表2 from 表1
````

### 5.2 批量导入旧表

>  从表 1 中查询出数据，批量插入到表 2 对应字段。

````sql
insert into #表2(字段1,字段2,...) select 字段1,字段2,... from 表1;
````

## 6. 临时表和表变量

> SQL Sever 中可以在一次连接中定义 **临时表**，或者 **表变量** 来存放一些临时数据。

### 6.1 临时表

> 创建语法和定义普通的表相同，但临时表表名必须以 # 开头。采用 ## 开头，表示定义成全局临时表。

````sql
CREATE TABLE #表名 (
	字段 数据类型 约束
	...
)
````

### 6.2 表变量

> 创建语法和定义普通变量类似，数据类型为 TABLE ，区别在于在数据类型后需要写上表字段结构

````sql
DECLARE @表名 TABLE (
	字段 数据类型	-- 表变量不能创建锁引和约束
	...
)
````

### 6.3 二者区别

* **创建方式**

    > 二者创建的定义方式不同。
    >
    > 且表变量不支持 批量导入新表 方式创建。

* **作用域**

    > **临时表**的作用域是在创建它们的 session 内。在同一 session 内，任何一个数据库的用户都可以查询和修改临时表。

    > **表变量**的作用域是在定义它们的批处理或者存储过程内。在这些对象内，只有定义它们的批处理或者存储过程可以使用表变量。

* **存储位置**

    > 临时表会被存储在 tempdb 数据库中，使用时会在内存中创建临时表。

    > 表变量只在内存中创建，不在磁盘上存储。

* **事务支持**

    > 临时表支持事务，可以使用 COMMIT 和 ROLLBACK 语句来提交或者撤销修改。

    > 表变量不能在事务中使用。

* **索引支持**

    > 时表可以创建索引和约束来提高查询和修改效率。

    > 表变量不能创建索引和约束。

## 7. 行列转换

### 7.1 行转列

````sql
-- 测试数据
CREATE TABLE [dbo].[#RowTableScore](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[UserName] [nvarchar](50) NULL,
	[Subject] [nvarchar](50) NULL,
	[Source] [numeric](18, 0) NULL
) ON [PRIMARY]
GO
INSERT INTO [#RowTableScore] ([UserName],[Subject],[Source])
	SELECT N'张三',N'语文',60 UNION ALL
	SELECT N'李四',N'数学',70 UNION ALL
	SELECT N'王五',N'英语',80 UNION ALL
	SELECT N'王五',N'数学',75 UNION ALL
	SELECT N'王五',N'语文',57 UNION ALL
	SELECT N'李四',N'语文',80 UNION ALL
	SELECT N'张三',N'英语',100
GO
SELECT * FROM [#RowTableScore]
````

#### 7.1.1 聚合函数

> 注：聚合函数 和 ELSE 后面数值的选择，得根据实际业务情况进行选择。

````sql
-- 静态写法。我们知道Subject列只有语文、数学、英语三个值，所以这里可以在 IN 后面可以手动列出来
SELECT UserName,
	Max(CASE [Subject] WHEN '数学' THEN Source ELSE 0 END) AS 数学,
	Max(CASE [Subject] WHEN '语文' THEN Source ELSE 0 END) AS 语文,
	Max(CASE [Subject] WHEN '英语' THEN Source ELSE 0 END) AS 英语
FROM #RowTableScore 
GROUP BY UserName
````

````sql
-- 动态写法。不知道Subject列有什么数据时，可用 SQL 拼接方式从表中查询并拼接进行查询。
DECLARE @sql AS NVARCHAR(MAX);

SELECT @sql =
	N'SELECT UserName,'+
	STRING_AGG(N'Max(CASE [Subject] WHEN '''+Subject+''' THEN Source ELSE 0 END) AS ['+[Subject]+']',',')+
	N' FROM #RowTableScore'+ -- 注意保留空格
	N' GROUP BY UserName' -- 注意保留空格
FROM
	(SELECT DISTINCT [Subject] FROM #RowTableScore) t

SELECT @sql
EXEC sp_executesql @sql;
````

#### 7.1.2 PIVOT

````sql
-- 语法
SELECT [非聚合列], [聚合列1], [聚合列2], ...
FROM (SELECT [列1], [列2], [列n] FROM [表]) t
PIVOT ([聚合函数]([聚合列]) FOR [非聚合列] IN ([列1], [列2], [列n])) p
````

````sql
-- 静态写法
SELECT *
FROM (SELECT [UserName] ,[Subject] ,[Source] FROM [#RowTableScore]) t
PIVOT (SUM([Source]) FOR [Subject] IN ( [数学],[英语],[语文] )) AS p
````

````sql
-- 动态写法
DECLARE @sql AS NVARCHAR(MAX);

SELECT @sql = 
  N'SELECT [UserName], ' + STRING_AGG('[' + [Subject] + ']', ', ') + 
  N' FROM (SELECT [UserName], [Subject], [Source] FROM [#RowTableScore]) p' +
  N' PIVOT (SUM([Source]) FOR [Subject] IN ('+ STRING_AGG('[' + [Subject] + ']', ', ')+ N')) AS pvt' 
FROM (
    SELECT DISTINCT [Subject] FROM [#RowTableScore]
) AS t;

EXEC sp_executesql @sql;
````

### 7.2 列转行

````sql
-- 测试数据
CREATE TABLE #ColTableScore
(
	Id INT primary KEY IDENTITY,
	[Name] VARCHAR(30),
	语文 INT,
	数学 INT,
	英语 INT
);
INSERT INTO #ColTableScore
VALUES 
	('张三',90,80,70),
	('李四',91,82,72),
	('王五',93,83,73),
	('赵六',94,84,74)
````

#### 7.2.1 UNION ALL

````sql
SELECT ID,Name,'语文' AS 科目,语文 AS 成绩 FROM #ColTableScore 
UNION ALL
SELECT ID,Name,'数学' AS 科目,数学 AS 成绩 FROM #ColTableScore 
UNION ALL
SELECT ID,Name,'英语' AS 科目,英语 AS 成绩 FROM #ColTableScore 
````

#### 7.2.2 UNPIVOT

````sql
SELECT ID,Name,科目,成绩 
FROM #ColTableScore
UNPIVOT(成绩 FOR 科目 IN(语文,数学,英语)) pt;
````

#### 7.2.3 CROSS APPLY

````SQL
SELECT Id, Name,科目,成绩
FROM [#ColTableScore]
CROSS APPLY (
    VALUES ('语文', 语文), ('数学', 数学), ('英语', 英语)
) AS ca(科目, 成绩);
````
