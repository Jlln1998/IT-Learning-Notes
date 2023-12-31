# SQL Sever 索引

## 1. 定义

> 将结构化数据中的一部分信息提取出来，重新组织，使其变得有一定数据结构，我们将这部分信息称之为索引。
>
> 通俗地讲，索引就相当于是书籍的目录，可以帮助我们快速寻找到数据，提高查询速度。

## 2. 索引的分类

> 索引主要分为两类：聚集索引（Clustered Index）、非聚集索引（Non-clustered Index）。

主要区别如下：

* 数据组织方式（核心区别）

    > 在聚集索引中，表中的数据行依据索引键值的顺序直接存储，因此索引和数据是紧密结合在一起的。
    >
    > 非聚集索引则会创建一个与实际数据表分开的单独索引结构，并在该结构中存储行数据的指针，这意味着非聚集索引和实际数据分离。

    > 注：该区别是两种索引的核心区别，后续的一些区别都是由此衍生出来的。

* 索引数量

    > 聚集索引每张表只能有一个，因为数据行实际的物理存储只能以一种方式进行排序。默认主键字段索引为聚集索引。
    >
    > 非聚集索引数量不限，因为非聚集索引是单独创建索引结构，用指针指向实际的数据行。

    > 注：主键一定是聚集索引，聚集索引并不一定是主键，在建表的时候，并没有加主键，这个时候如果说建立了一个聚集索引，再建立主键，那么这个时候 主键就不是聚集索引了。

* 数据物理排序：

    >  聚集索引确定了表中数据行的物理排序，即数据行依据聚集索引键值进行排序。
    >
    > 非聚集索引仅依据索引键值进行排序，而与表中数据行的物理顺序无关。

* 数据查询访问效率：

    > 聚集索引由于实际数据已经和索引键值顺序保持一致，能够更快地访问数据，特别是顺序查询和范围查询。
    >
    > 非聚集索引需要先在索引结构中查找相应索引键值，再通过指针引用访问实际数据，因此查询过程中可能涉及更多的I/O操作，其访问效率相对较低。

* 数据增、删、盖操作的影响：

    > 聚集索因为会引影响数据行的物理顺序，当表中的数据发生增、删、改时，需要调整聚集索引，性能开销大，效率可能会较慢。
    >
    > 非聚集索引对这些操作的影响较小，因为它们不直接影响数据行的物理顺序。

## 3. 索引的数据结构

> SQL Sever 中索引的数据结构是 B+ 树。

索引为什么选择B+树？

* 平衡性：

    > B+ 树是一个平衡的多路搜索树，所有叶子节点处于相同的高度。这保证了查询操作在最坏情况下的性能。
    >
    > 对于大型数据库，B+ 树结构能够保持较低的树高，从而使得查找、插入、删除等操作的时间复杂度较低。

* 数据存储：

    > B+ 树中的叶子节点存放所有数据记录，而内部节点仅存储键值，并引导搜索方向。这使得每个内部节点可以存储更多的键值，提高了树形结构的空间利用率和扩展性。

* 顺序访问性：

    > B+ 树中的叶子节点通过指针相互链接，这使得整个数据集能够按排序顺序进行顺序访问。这种顺序访问能力非常适合应对范围查询和排序操作。

* 磁盘友好：

    > 由于 SQL Server 索引的数据通常存储在磁盘上，B+ 树具有很好的磁盘友好性。
    >
    > B+ 树可以有效减少磁盘 I/O 操作次数，提高访问效率。
    >
    > 在实际应用中，B+ 树允许每个节点容纳的键值与磁盘页大小相匹配，这样每次磁盘 I/O 可以加载一个节点的数据，从而最大限度地减少磁盘访问次数。

* 动态增长和缩小：

    > B+ 树在插入和删除元素时，能够通过节点分裂和合并来灵活调整其结构，维持树的平衡性，确保查询性能稳定。

综上所述，B+ 树以其优异的平衡性、高效的数据存储、顺序访问性、磁盘友好性和动态调整能力，成为 SQL Server 索引的理想结构。



## 4. 索引设计原则

索引不是越多越好：

> * 索引也会占用空间。
>
> * 索引页有需要系统维护，每次增删改数据都可能对索引进行重新编排。
>
> * 索引堆积久了会产生索引碎屏，反而不利于数据查询。

建议建立索引的列：

* 主键列（默认会有聚集索引）
* 外键列
* 经常查询的列、以及经常在order by、group by、distinct 等关键字后面进行判断的列。
* 重复值比较多的列不能建立索引，例如：性别。
* 对于 text,image,bit 这些数据类型的字段不能建立索引。
* 经常存取的列不要建立索引。

## 5. 使用索引

### 5.1 创建索引

````sql
CREATE [UNIQUE] [CLUSTERED / NONCLUSTERED]
INDEX 自定义索引名
on 表名(列名1,列名2, …)
````

> UNIQUE：可选，表示创建唯一索引。
>
> CLUSTERED：表示创建聚集索引
>
> NONCLUSTERED：表示创建非聚集索引，不写明的话，默认表示创建非聚集索引。
>
> 索引名称规范：`index_表名_字段名_自定义序号标识`

### 5.2 查看索引

> 查看一个表的所有索引情况

````sql
exec sp_helpindex '表名'
````

### 5.3 重命名索引

````sql
exec sp_rename '表名.旧索引名','新索引名','index';
````

### 5.4 删除索引

````sql
DROP INDEX 索引名 on 表名
````

### 5.5 重建索引

````sql
ALTER INDEX 索引名称 on 数据表名 REBUILD
````