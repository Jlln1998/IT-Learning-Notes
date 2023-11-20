# SQL Sever 表定义操作

## 1. GO 语句

* go 向 SQL Server 实用工具发出一批 Transact-SQL 语句结束的信号。

* go 是把 T-sql 语句分批次执行。（一步成功了才会执行下一步，即一步一个go）。

* 每个被 go 分隔的语句都是一个单独的事务，一个语句执行失败不会影响其它语句执行。

* go 语句后可以跟数字，代表该批 T-SQL 执行次数。

<img src="./assets/GO 批处理执行语句.png" alt="GO 批处理执行语句" style="zoom:50%;" />

## 2. 表的创建与修改

### 2.1 创建表

````SQL
create table 表名(
	字段1 数据类型,
    字段2 数据类型,
    [...,]
    字段n 数据类型
)
````

### 2.2 添加字段（列）

````sql
ALTER TABLE 表名 ADD 字段 数据类型;

-- 批量添加字段
ALTER TABLE 表名 ADD
字段1 数据类型,
字段2 数据类型
````

### 2.3 修改字段数据类型

````sql
ALTER TABLE 表名 ALTER COLUMN 字段 数据类型;
````

### 2.4 重命名

````sql
-- 表重命名
exec sp_rename'旧表名', '新表名'

-- 表字段重命名
exec sp_rename'表名.旧字段名','新字段名','column'
````

### 2.5 删除字段

```sql
ALTER TABLE 表名 DROP COLUMN 字段 
```

### 2.6 删除表

````sql
DROP TABLE 表名
````
