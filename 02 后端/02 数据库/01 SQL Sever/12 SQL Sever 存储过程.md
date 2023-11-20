# SQL Sever 存储过程

## 1. 定义

> 一系列 T-SQL 语句的集合，用于封装复杂的 SQL 语句。
>
> 优点：可以减少网络通讯、提高程序执行速度与复用性、提高安全性。
>
> SQL Sever 存储过程分为 系统存储过程 和 自定义存储过程。

## 2. 系统存储过程

> 常用的系统存储过程如下

| 存储过程              | 作用                             |
| --------------------- | -------------------------------- |
| sys.sp_databases      | 查看当前服务器所有数据库以及大小 |
| sys.sp_tables         | 查询数据库中的表                 |
| sys.sp_columns        | 查询指定表的列字段信息           |
| sys.sp_helpindex      | 查询指定表的索引信息             |
| sys.sp_helpconstraint | 查询指定表的约束信息             |
| sp_rename             | 修改表、索引、列的名称           |

## 3. 自定义存储过程

````sql
CREATE PROCEDURE [dbo].[数据库名]
	@参数1 数据类型1,
	...
	@参数n 数据类型n
AS
BEGIN
	SET NOCOUNT ON;
	-- 逻辑语句
END
GO
````



