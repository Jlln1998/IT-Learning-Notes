# SQL Sever 表数据操作

## 1. 新增数据

### 1.1 基本语法

````sql
-- 单条插入
INSERT INTO 表名(字段1,字段2,字段3,字段4....)
VALUES(数据1,数据2,数据3,数据4...)

-- 批量插入
INSERT INTO 表名(字段1,字段2,字段3,字段4....)
VALUES
(数据1,数据2,数据3,数据4...),
(数据1,数据2,数据3,数据4...),
(数据1,数据2,数据3,数据4...),
.....;
````



## 2. 删除数据

### 2.1 delete

> 一条一条地删除数据

````sql
DELETE FROM 表名 WHERE 筛选条件
````

### 2.2 清空表

* 方式一：使用不加条件的 DELETE 语句。

    > 原理：一条条地删除表中所有数据
    >
    > 特点：对于数据多的表速度较慢，并且不会重置自增主键的计数，但是可以回滚数据。

    ````sql
    DELETE FROM 表名
    ````

* 方式二：TRUNCATE

    > 原理：直接把整个表删除，再新建一个和原来结构一模一样的表。
    >
    > 特点：速度快、并且会重置自增主键的计数，不能回滚数据。

    ````sql
    TRUNCATE TABLE 表名
    ````



## 3. 修改数据

### 3.1 基本语法

````sql
UPDATE 表名 
SET 字段1=值1,字段2=值2,字段3=值3,... 
WHERE 筛选条件
````



## 4. 查询数据

### 4.1 基本查询

### 4.2 去重

> 关键字：DISTINCT

````sql
SELECT 字段 FROM 表名
````

### 4.3 查询前面部分数据

````sql
SELECT TOP (100) * FROM  表名 -- 查询前100条数据
SELECT TOP 30 percent * FROM  表名 -- 查询前 30% 条数据,向上取整。
````

### 4.4 排序

> ASC：升序排序（默认）
>
> DESC：降序排序

````sql
-- 单字段排序
SELECT 字段1,字段2,字段3,.... 
FROM 表名
WHERE 筛选条件
ORDER BY 字段1 DESC

-- 多字段排序：先按字段1降序排序，如果字段1有相同数据，那么相同数据间再按字段2升序排序.
SELECT 字段1,字段2,字段3,.... 
FROM 表名
WHERE 筛选条件
ORDER BY 字段1 DESC,字段2 ASC
````

### 4.5 between....and...

> 表示一个区间的数据，区间是闭区间

````sql
SELECT 字段1,字段2,字段3,.... 
FROM 表名
WHERE 字段1 BETWEEN 10 AND 20
````

## 5. 聚合函数

> 聚合函数的括号中，也可配合DISTINCT关键字使用

| 函数    | 作用         |
| ------- | ------------ |
| MIN()   | 取最小值     |
| MAX()   | 取最大值     |
| SUM()   | 求和         |
| COUNT() | 求总数据条数 |
| AVG()   | 求平均数     |

## 6. 分组查询

````sql
SELECT 字段1,字段2,字段3 
FROM 表名 
GROUP BY 字段1
HAVING 分组后的筛选条件
````

## 7. 嵌套查询

> 嵌套查询又称子查询，将一段 SELECT 查询语句的结果当作另一个 SELECT 查询语句的值或者集合。

```sql
SELECT * FROM 表名 WHERE 字段 运算符 (SELECT 子语句)
```

### 7.1 any() & some()

> * **any** 比子集中任意一个数据...
> * **some** 比子集中的某个数据...，与 any 等价

````sql
SELECT * FROM 表名 WHERE 字段 > any (SELECT 子语句)
SELECT * FROM 表名 WHERE 字段 > some (SELECT 子语句)
````

### 7.2 all()

> * **all** 比子集中所有数据都...

````SQL
SELECT * FROM 表名 WHERE 字段 > all (SELECT 子语句)
````

### 7.3 in()

> * **in** 等于子集中的某个数据

````sql
SELECT * FROM 表名 WHERE 字段 in (SELECT 子语句)
````

### 7.4 exists()

> * **exists** 子集中能查询出数据即为真。

````SQL
SELECT * FROM 表名 WHERE exists (SELECT 子语句)
````

## 8. 分页查询

### 8.1 Row_Number()

> 函数：Row_Number() Over(Order by 字段 asc/desc ) 用于给结果按指定排序规则增加序号列。

````sql
SELECT column1, column2, ...
FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY 字段) AS rowNum,column1, column2, ...
    FROM 表名
) AS 别名
WHERE rowNum BETWEEN (pageNumber - 1) * pageSize + 1 AND pageNumber * pageSize;
````

### 8.2 Offset Fetch

> SQL Sever 2012 及以上版本适用。

````sql
SELECT column1, column2, ...
FROM 表名
ORDER BY column
OFFSET (pageNumber - 1) * pageSize ROWS
FETCH NEXT pageSize ROWS ONLY;
````

## 9. 表连接查询

### 9.1 内连接

> 语法：表A join 表B on 连接条件
>
> 作用：把两个表相互匹配的数据关联成一张结果表进行查询。两张表各自未匹配的数据都不会显示。
>
> 注：join 也可以写成 inner join

### 9.2 外连接

#### 9.2.1 左外连接

> 语法：表A left join 表B on 连接条件
>
> 作用：以左表为基表，返回左表全部数据，以及符合匹配条件的右表数据，不符合匹配条件的字段用null填充。

#### 9.2.2 右外连接

> 语法：表A right join 表B on 连接条件
>
> 作用：以右表为基表，返回右表全部数据，以及符合匹配条件的左表数据，不符合匹配条件的字段用null填充。

#### 9.2.2 全外连接

> 语法：表A outer join 表B on 连接条件
>
> 作用：左右表全部数据都会返回，不符合匹配条件的字段用null填充。

### 9.3 不等值连接

> 连接条件一般是使用 = 号，也可以使用>、<、!= 等等符号，称为不等值连接。

### 9.4 自连接

> 一个表与自身进行表连接查询，常用于菜单表的查询（Id = ParentId）。

## 10. 联合查询

### 10.1 union

> 将 2 个 SELECT 结果集合并成 1 个结果集，并且去除重复数据。
>
> 注意：2 个 SELECT 结果集的列数、以及数据类型要相同。

````sql
SELECT 字段 FROM 表1
UNION
SELECT 字段 FROM 表2
````

### 10.2 union all

> 将两个 SELECT 结果集合并成 1 个结果集，不会去重。

````SQL
SELECT 字段 FROM 表1
UNION ALL
SELECT 字段 FROM 表2
````
