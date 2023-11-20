# SQL Sever 执行计划

## 1. 概念

官方文档：[SQL Sever 执行计划概念](https://learn.microsoft.com/zh-cn/sql/relational-databases/performance/execution-plans?view=sql-server-ver16)

> SQL Sever 执行 SQL 查询语句时，数据库引擎的查询优化器的组件，会先根据查询语句、表和索引的定义、数据库设计以及数据库统计信息等数据，经过分析生成一个或多个查询执行计划。
>
> 然后查询优化器使用一组启发式方法来选择查询计划，以平衡编译时间和计划优化，从而选择出最优的查询计划。

查询计划会确定下列事情：

* **访问源表的顺序**

> 例如，一个查询语句涉及了A、B、C三个表，获取数据的顺序可能有ABC、BCA、CAB....等等多中顺序。执行计划会根据 SQL 语句分析出查询的顺序。

* **从每个表提取数据的方法**

> 访问每个表的数据也有多中方式，例如
>
> * 如果只需要有特定键值的几行，数据库服务器可以使用**索引**。
> * 如果需要表中的所有行，数据库服务器则可以忽略索引并执行**表扫描**。
> * 如果需要表中的所有行，而有一个索引的键列在 `ORDER BY`中，则执行索引扫描而非表扫描。可能会省去对结果集的单独排序
> * 如果表很小，则对该表的几乎所有访问来说，表扫描可能都是最有效的方法。

* **用于计算的方法，以及如何对每个表中的数据进行筛选、聚合和排序的方法。**

> 从表访问数据时，可以使用不同的方法对数据进行计算。
>
> 例如，计算标量值，以及对查询文本中定义的数据进行聚合和排序（例如，使用 `GROUP BY` 或 `ORDER BY` 子句时），以及如何筛选数据（例如在使用 `WHERE` 或 `HAVING` 子句时）



## 2. 执行计划相关术语解释

> 

| 术语                         | 介绍                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 自适应联接                   | 通过“自适应联接”运算符，在扫描第一个输入后可延迟选择哈希联接或嵌套循环联接方法。 “自适应联接”运算符是一个物理运算符。 有关详细信息，请参阅[理解自适应联接](https://learn.microsoft.com/zh-cn/sql/relational-databases/performance/joins?view=sql-server-ver16#adaptive)。 |
| 聚合                         | **Aggregate** 运算符计算包含 MIN、MAX、SUM、COUNT 或 AVG 的表达式。 **Aggregate** 既是一个逻辑运算符，也是一个物理运算符。 |
| Arithmetic Expression        | **Arithmetic Expression** 运算符根据行中的现有值计算新值。 较新版本的 SQL Server 不使用**算术表达式**。 |
| Async Concat                 | Async Concat 运算符仅用于远程查询中（分布式查询）。 它有 *n* 个子节点和一个父节点。 通常，某些子节点是参与分布式查询的远程计算机。 Asnyc Concat 同时向所有子节点发出 `open()` 调用，然后将位图应用于每个子节点。 对于为 1 的每个位， **Async Concat** 按需向父节点发送输出行。 |
| ActualRebinds                | **Assert** 运算符用于验证条件。 例如，验证引用完整性或确保标量子查询返回一行。 对于每个输入行， **Assert** 运算符都要计算执行计划的 **Argument** 列中的表达式。 如果此表达式的值为 NULL，则通过 **Assert** 运算符传递该行，并且查询执行将继续。 如果此表达式的值非 Null，则将产生相应的错误。 **Assert** 运算符是一个物理运算符。 |
| Assign                       | **Assign** 运算符将表达式的值或常量分配给变量。 **Assign** 是一个语言元素。 |
| Bitmap Create                | **Bitmap Create** 运算符出现在创建位图的显示计划输出中。 **Bitmap Create** 是一个逻辑运算符。 |
| Bitmap                       | SQL Server 使用 **Bitmap** 运算符来实现并行查询计划中的位图筛选。 在将行传递给另一个运算符（如 **Parallelism** 运算符）之前，通过消除无法生成任何联接记录的键值的行，位图筛选可提高查询的执行速度。 位图筛选器使用运算符树某部分的表中一组值的简洁表示形式来筛选位于该树另一部分的第二张表中的行。 通过在查询中预先删除不必要的行，后续运算符将处理较少的行，从而提高查询的整体性能。 优化器将确定位图的选择性何时可满足使用条件以及在哪些运算符上应用筛选器。 **Bitmap** 是一个物理运算符。 |
| Bookmark Lookup              | **Bookmark Lookup** 运算符使用书签（行 ID 或聚集键）在表或聚集索引中查找相应的行。 **Argument** 列包含书签标签，用于在表或聚集索引内查找行。 **Argument** 列还包含要查找的行所在的表或聚集索引的名称。 如果 **Argument** 列中出现 WITH PREFETCH 子句，则表示查询处理器已决定在表或聚集索引中查找书签时将使用异步预提取（预读）作为最佳选择。  从 SQL Server 2005 (9.x) 开始，便不再使用 Bookmark Lookup。 而由 Key Lookup 和 RID Lookup 提供书签查找功能 。 |
| Branch Repartition           | 在并行查询计划中，有时存在迭代器的概念性区域。 此类区域中的所有迭代器都可通过并行线程执行。 这些区域本身必须串行执行。 单个区域内的某些 **Parallelism** 迭代器称为 **Branch Repartition**。 两个这样的区域边界上的 **Parallelism** 迭代器称为 **Segment Repartition**。 **Branch Repartition** 和 **Segment Repartition** 是逻辑运算符。 |
| Broadcast                    | **Broadcast** 有一个子节点和 *n* 个父节点。 **Broadcast** 根据使用者的请求将其输入行发送给多个使用者。 每个使用者都将获得所有行。 例如，如果所有使用者都是哈希联接的生成端，则将生成 *n* 份哈希表。 |
| Build Hash                   | 指示为 xVelocity 内存优化的列存储索引生成批处理哈希表。      |
| 缓存                         | **Cache** 是一个专门的 **Spool** 运算符。 它仅存储一行数据。 **Cache** 是一个逻辑运算符。 **较**新版本的 SQL Server 中不使用缓存。 |
| Clustered Index Delete       | **Clustered Index Delete** 运算符从查询执行计划的 Argument 列中指定的聚集索引中删除行。 如果 Argument 列中存在 WHERE:() 谓词，则仅删除满足该谓词的行。**Clustered Index Delete** 是一个物理运算符。 |
| Clustered Index Insert       | **Clustered Index Insert** Showplan 运算符可将行从其输入插入到 Argument 列中所指定的聚集索引中。 Argument 列还包含一个 SET:() 谓词，用于指示为每一列设置的值。 如果 **Clustered Index Insert** 的插入值没有子项，则插入的行来自 **Insert** 运算符本身。**Clustered Index Insert** 是一个物理运算符。 |
| Clustered Index Merge        | **Clustered Index Merge** 运算符可将合并数据流应用于聚集索引。 该运算符可在其 **Argument** 列中所指定的聚集索引中删除、更新或插入行。 执行的实际操作取决于该运算符的 **Argument** 列中指定的 **ACTION** 列的运行时值。 **Clustered Index Merge** 是一个物理运算符。 |
| Clustered Index Scan         | **Clustered Index Scan** 运算符会扫描查询执行计划的 Argument 列中指定的聚集索引。 存在可选 WHERE:() 谓词时，则只返回满足该谓词的那些行。 如果 Argument 列包含 ORDERED 子句，则表示查询处理器已请求按聚集索引排列行的顺序返回行输出。 如果没有 ORDERED 子句，存储引擎将以最佳方式扫描索引，而无需对输出进行排序。 **Clustered Index Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Clustered Index Seek         | **Clustered Index Seek** 运算符可以利用索引的查找功能从聚集索引中检索行。 **Argument** 列包含所使用的聚集索引名称和 SEEK:() 谓词。 存储引擎仅使用索引来处理满足此 SEEK:() 谓词的行。 它还包括 WHERE:() 谓词，其中存储引擎对满足 SEEK:() 谓词的所有行进行计算，但此操作是可选的，并且不使用索引来完成此过程。  如果 **Argument** 列包含 ORDERED 子句，则表示查询处理器已决定必须按聚集索引排序行的顺序返回行。 如果没有 ORDERED 子句，存储引擎将以最佳方式搜索索引，而不对输出进行必要的排序。 若允许输出保持顺序，则效率可能比生成非排序输出的效率低。 出现关键字 LOOKUP 时，将执行书签查找。 在 SQL Server 2008 (10.0.x) 及更高版本中，**键查找**运算符提供书签查找功能。 **Clustered Index Seek** 既是一个逻辑运算符，也是一个物理运算符。 |
| Clustered Index Update       | **Clustered Index Update** 运算符更新 **Argument** 列中指定的聚集索引中的输入行。如果存在 WHERE:() 谓词，则只更新那些满足此谓词的行。 如果存在 SET:() 谓词，则将每个更新的列设置为该值。 如果存在 DEFINE:() 谓词，则列出此运算符定义的值。 可以在 SET 子句中、该运算符内的其他位置和该查询内的其他位置引用这些值。 **Clustered Index Update** 既是一个逻辑运算符，也是一个物理运算符。 |
| 折叠                         | **Collapse** 运算符用于优化更新处理。 执行更新时，可以将该更新操作拆分（使用 **Split** 运算符）成为删除和插入操作。 **Argument** 列包含一个指定键列列表的 GROUP BY:() 子句。 如果查询处理器遇到删除和插入相同键值的相邻行，则会用一个更有效的更新操作替换这些单独的操作。 **Collapse** 既是一个逻辑运算符，也是一个物理运算符。 |
| Columnstore Index Scan       | **Columnstore Index Scan** 运算符会扫描查询执行计划的 **Argument** 列中指定的列存储索引。 |
| Compute Scalar               | **Compute Scalar** 运算符通过对表达式求值来生成计算标量值。 该值可以返回给用户、在查询中的其他位置引用或二者皆可。 例如，在筛选谓词或联接谓词中就会出现二者皆可的情况。 **Compute Scalar** 既是一个逻辑运算符，也是一个物理运算符。  在 SET STATISTICS XML 生成的显示计划中出现的**Compute Scalar** 运算符可能不包含 **RunTimeInformation** 元素。 在图形显示计划中，当已在 **中选中**“包括实际的执行计划” **选项时，** “实际行” **、** “实际重新绑定次数” **和** “实际重绕次数” **可能不会出现在** “属性” SQL Server Management Studio窗口中。 当出现这种情况时，意味着虽然编译过的查询计划中使用了这些运算符，但在运行时查询计划中，它们的作用是由其他运算符实现的。 另外，请注意，SET STATISTICS PROFILE 生成的显示计划输出中的执行数等于 SET STATISTICS XML 生成的显示计划中的重新绑定次数和重绕次数的总和。 |
| Concatenation                | **Concatenation** 运算符扫描多个输入，并返回每个扫描的行。 **串联** 通常用于实现 Transact-SQL UNION ALL 构造。 **Concatenation** 物理运算符有两个或多个输入，有一个输出。 Concatenation 将行从第一个输入流复制到输出流，然后对其他输入流重复进行此操作。 **Concatenation** 既是一个逻辑运算符，也是一个物理运算符。 |
| Constant Scan                | **Constant Scan** 运算符可将一个或多个常量行引入到查询中。 **Compute Scalar** 运算符通常在 **Constant Scan** 之后使用，以将列添加到 **Constant Scan** 运算符生成的行中。 |
| 转换                         | **Convert** 运算符将标量数据类型转换为另一种类型。 **Convert** 是一个语言元素。 |
| Cross Join                   | **Cross Join** 运算符将第一个（顶端）输入中的每一行与第二个（底端）输入中的每一行联接在一起。 **Cross Join** 是一个逻辑运算符。 |
| 游标                         | **Cursor** 逻辑运算符和物理运算符用于描述涉及游标操作的查询或更新的执行方式。 其中物理运算符描述用于处理游标（如使用键集驱动游标）的物理实现算法。 游标执行过程的每一步都涉及物理运算符。 而逻辑运算符描述游标的属性，如游标是只读。  逻辑运算符包括 Asynchronous、Optimistic、Primary、Read Only、Scroll Locks、Secondary 和 Synchronous。  物理运算符包括 Dynamic、Fetch Query、Keyset、Population Query、Refresh Query 和 Snapshot。 |
| catchall                     | 生成图形显示计划的逻辑找不到迭代器的合适图标时，将显示通用图标。 通用图标不一定指示存在错误。 有三个可捕获图标：用于迭代器) 的蓝色 (、游标) 的橙色 (和 Transact-SQL 语言元素) 的绿色 (。 |
| Declare                      | Declare 运算符用于分配查询计划中的局部变量。 **Declare** 是一个语言元素。 |
| 删除                         | **Delete** 运算符将从对象中删除满足 **Argument** 列内的可选谓词的行。 |
| Deleted Scan                 | **Deleted Scan** 运算符在触发器中扫描删除的表。              |
| Distinct Sort                | **Distinct Sort** 逻辑运算符将对输入进行扫描，删除重复项并按 **Argument** 列的 DISTINCT ORDER BY:() 谓词中指定的列进行排序。 **Distinct Sort** 是一个逻辑运算符。 |
| Distinct                     | **Distinct** 运算符可以从行集或值集中删除重复项。 **Distinct** 是一个逻辑运算符。 |
| Distribute Streams           | **Distribute Streams** 运算符仅用于并行查询计划。 **Distribute Streams** 运算符接收记录的单个输入流，并生成多个输出流。 记录的内容和格式不会改变。 输入流中的每个记录都将在某个输出流中显示。 此运算符在输出流中自动保留输入记录的相对顺序。 通常情况下，使用哈希操作确定特定输入记录所属的输出流。  如果将输出分区，那么 **Argument** 列会包含 PARTITION COLUMNS:() 谓词和分区依据列。 **Distribute Streams** 是一个逻辑运算符。 |
| Dynamic                      | **Dynamic** 运算符使用可以查看其他游标所做的任何更改的游标。 |
| Fetch Query                  | 当对游标发出提取命令时， **Fetch Query** 运算符将检索行。    |
| 筛选器                       | **Filter** 运算符扫描输入，仅返回那些符合 **Argument** 列中的筛选表达式（谓词）的行。 |
| Flow Distinct                | **Flow Distinct** 逻辑运算符用于通过扫描输入来删除重复项。 虽然 **Distinct** 运算符在生成任何输出前使用所有的输入，但 **FlowDistinct** 运算符在从输入获得行时会返回每行（除非该行是一个重复项，若是这样则删除该行）。 |
| Foreign Key References Check | Foreign Key References Check 运算符通过将修改行与引用表中的行进行比较，就地执行引用完整性检查，以便验证修改不会中断引用完整性。 当相同主键或唯一键上存在超过 253 个外键引用时，将使用 Foreign Key References Check 运算符。 Foreign Key References Check 是逻辑和物理运算符。 |
| Full Outer Join              | **Full Outer Join** 逻辑运算符从第一个（顶端）输入与第二个（底端）输入相联接的行中返回每个满足联接谓词的行。 它还可以从下列输入返回行：  \- 在第二个输入中没有匹配项的第一个输入。  \- 在第一个输入中没有匹配项的第二个输入。  不包含匹配值的输入将作为空值返回。 **Full Outer Join** 是一个逻辑运算符。 |
| Gather Streams               | **Gather Streams** 运算符仅用在并行查询计划中。 **Gather Streams** 运算符处理几个输入流并通过组合这几个输入流生成单个记录输出流。 记录的内容和格式不会改变。 如果此运算符保留顺序，则所有的输入流都必须有序。 如果输出已排序，则 **Argument** 列会包含一个 ORDER BY:() 谓词和正在排序的列名称。 **Gather Streams** 是一个逻辑运算符。 |
| Hash Match                   | **Hash Match** 运算符通过计算其生成输入中每行的哈希值生成哈希表。 HASH:() 谓词以及一个用于创建哈希值的列的列表会出现在 **Argument** 列中。 然后，该谓词为每个探测行（如果适用）计算哈希值（使用相同的哈希函数）并在哈希表内查找匹配项。 如果存在残留谓词（由 **Argument** 列中的 RESIDUAL:() 标识），则还须满足此残留谓词，只有这样行才能被视为是匹配项。 行为取决于所执行的逻辑操作：  \- 对于联接，使用第一个（顶端）输入生成哈希表，使用第二个（底端）输入探测哈希表。 按联接类型规定的模式输出匹配项（或不匹配项）。 如果多个联接使用相同的联接列，这些操作将分组为一个哈希组。  \- 对于非重复或聚合运算符，使用输入生成哈希表（删除重复项并计算聚合表达式）。 生成哈希表时，扫描该表并输出所有项。  \- 对于 union 运算符，使用第一个输入生成哈希表（删除重复项）。 使用第二个输入（它必须没有重复项）探测哈希表，返回所有没有匹配项的行，然后扫描该哈希表并返回所有项。 **Hash Match** 是一个物理运算符。 有关详细信息，请参阅[理解哈希联接](https://learn.microsoft.com/zh-cn/sql/relational-databases/performance/joins?view=sql-server-ver16#hash)。 |
| If                           | **If** 运算符执行基于表达式的有条件处理。 **If** 是一个语言元素。 |
| Inner Join                   | **Inner Join** 逻辑运算符返回满足联接第一个（顶端）输入与第二个（底端）输入的每一行。 |
| 插入                         | **Insert** 逻辑运算符将每行从其输入插入 **Argument** 列内指定的对象中。 相应的物理运算符为 **Table Insert**、 **Index Insert**或 **Clustered Index Insert** 运算符。 |
| Inserted Scan                | **Inserted Scan** 运算符扫描 **插入的** 表。 **Inserted Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Intrinsic                    | **内部**运算符调用内部 Transact-SQL 函数。 **Intrinsic** 是一个语言元素。 |
| 迭代器                       | 生成图形显示计划的逻辑找不到 **Iterator** 的合适图标时，将显示通用图标。 通用图标不一定指示存在错误。 有三个捕获图标：用于迭代器) 的蓝色 (、游标) 的橙色 (和 transact-SQL 语言构造) 的绿色 (。 |
| Key Lookup                   | **Key Lookup** 运算符是在具有聚集索引的表上进行的书签查找。 **Argument** 列包含聚集索引的名称和用来在聚集索引中查找行的聚集键。 **Key Lookup** 通常带有 **Nested Loops** 运算符。 如果 **Argument** 列中出现 WITH PREFETCH 子句，则表示查询处理器已决定在聚集索引中查找书签时将使用异步预提取（预读）作为最佳选择。  在查询计划中使用 **Key Lookup** 运算符表明该查询可能会从性能优化中获益。 例如，添加涵盖索引可能会提高查询性能。 |
| Keyset                       | **Keyset** 运算符使用的游标可用于查看其他用户所做的更新，而不能查看其他用户所做的插入。 |
| Language 元素                | 生成图形显示计划的逻辑找不到 **Language Element** 的合适图标时，将显示通用图标。 通用图标不一定指示存在错误。 有三个捕获图标：用于迭代器) 的蓝色 (、游标) 的橙色 (和 transact-SQL 语言构造) 的绿色 (。 |
| Left Anti Semi Join          | 当第二个（底端）输入中没有匹配行时， **Left Anti Semi Join** 运算符则返回第一个（顶端）输入中的每一行。 如果 **Argument** 列内不存在任何联接谓词，则每行都是一个匹配行。 **Left Anti Semi Join** 是一个逻辑运算符。 |
| Left Outer Join              | **Left Outer Join** 运算符返回满足联接第一个（顶端）输入与第二个（底端）输入的每一行。 它还返回任何在第二个输入中没有匹配行的第一个输入中的行。 第二个输入中的非匹配行作为空值返回。 如果 **Argument** 列内不存在任何联接谓词，则每行都是一个匹配行。 **Left Outer Join** 是一个逻辑运算符。 |
| Left Semi Join               | 当第二个（底端）输入中没有匹配行时， **Left Semi Join** 运算符则返回第一个（顶端）输入中的每一行。 如果 **Argument** 列内不存在任何联接谓词，则每行都是一个匹配行。 **Left Semi Join** 是一个逻辑运算符。 |
| Log Row Scan                 | **Log Row Scan** 运算符用于扫描事务日志。 **Log Row Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Merge Interval               | **Merge Interval** 运算符可合并多个（可能重叠的）间隔以得出最小的不重叠间隔，然后将其用于查找索引项。 此运算符通常出现在 **Constant Scan** 运算符中的一个或多个 **Compute Scalar** 运算符上方，Constant Scan 运算符构造了此运算符所合并的间隔（表示为一行中的列）。 **Merge Interval** 既是一个逻辑运算符，也是一个物理运算符。 |
| 合并联接                     | **Merge Join** 运算符执行内部联接、左外部联接、左半部联接、左反半部联接、右外部联接、右半部联接、右反半部联接和联合逻辑运算。  在 **Argument** 列中，如果操作执行一对多联接，则 **Merge Join** 运算符将包含 MERGE:() 谓词；如果操作执行多对多联接，则该运算符将包含 MANY-TO-MANY MERGE:() 谓词。 **Argument** 列还包含一个用于执行操作的列的列表，该列表以逗号分隔。 **Merge Join** 运算符要求在各自的列上对两个输入进行排序，这可以通过在查询计划中插入显式排序操作来实现。 如果不需要显式排序（例如，如果数据库内有合适的 B 树索引或可以对多个操作（如合并联接和对汇总分组）使用排序顺序），则合并联接尤其有效。 **Merge Join** 是一个物理运算符。 有关详细信息，请参阅[理解合并联接](https://learn.microsoft.com/zh-cn/sql/relational-databases/performance/joins?view=sql-server-ver16#merge)。 |
| Nested Loops                 | **Nested Loops** 运算符执行内部联接、左外部联接、左半部联接和左反半部联接逻辑运算。 嵌套循环联接通常使用索引，针对外部表的每一行在内部表中执行搜索。 查询处理器根据预计的开销来决定是否对外部输入进行排序，以改进内部输入索引上的搜索定位。 将基于所执行的逻辑操作返回所有满足 **Argument** 列中的（可选）谓词的行。 如果 OPTIMIZED 特性设置为“True”，则表示使用了优化的嵌套循环（或批处理排序）。 **Nested Loops** 是一个物理运算符。 有关详细信息，请参阅[了解嵌套循环联接](https://learn.microsoft.com/zh-cn/sql/relational-databases/performance/joins?view=sql-server-ver16#nested_loops)。 |
| Nonclustered Index Delete    | **Nonclustered Index Delete** 运算符通过 **Argument** 列中指定的非聚集索引删除输入行。 **Nonclustered Index Delete** 是一个物理运算符。 |
| Index Insert                 | **Index Insert** 运算符用于将行从其输入插入到 **Argument** 列中指定的非聚集索引中。 **Argument** 列还包含一个 SET:() 谓词，用于指示为每一列设置的值。 **Index Insert** 是一个物理运算符。 |
| Index Scan                   | **Index Scan** 运算符从 **Argument** 列中指定的非聚集索引中检索所有行。 如果可选的 WHERE:() 谓词出现在 **Argument** 列中，则仅返回满足此谓词的那些行。 **Index Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Index Seek                   | **Index Seek** 运算符利用索引的查找功能从非聚集索引中检索行。 **Argument** 列包含所使用的非聚集索引的名称。 它还包括 SEEK:() 谓词。 存储引擎仅使用索引来处理满足 SEEK:() 谓词的行。 它可能还包含一个 WHERE:() 谓词，其中存储引擎对满足 SEEK:() 谓词的所有行进行计算（不使用索引来完成）。 如果 **Argument** 列包含 ORDERED 子句，则表示查询处理器已决定必须按非聚集索引排序行的顺序返回行。 如果没有 ORDERED 子句，则存储引擎将以最佳方式（不保证对输出排序）搜索索引。 如果让输出保持其顺序，则效率可能低于生成非排序输出。 **Index Seek** 既是一个逻辑运算符，也是一个物理运算符。 |
| Index Spool                  | **Index Spool** 物理运算符在 **Argument** 列中包含 SEEK:() 谓词。 **Index Spool** 运算符扫描其输入行，将每行的副本放置在隐藏的假脱机文件（存储在 **tempdb** 数据库中且只在查询的生存期内存在）中，并在这些行上生成非聚集索引。 这样可以使用索引的查找功能来仅输出那些满足 SEEK:() 谓词的行。 如果重绕该运算符（例如通过 **Nested Loops** 运算符重绕），但不需要任何重新绑定，则将使用假脱机数据，而不用重新扫描输入。 |
| Nonclustered Index Update    | **Nonclustered Index Update** 物理运算符用于更新 **Argument** 列内指定的非聚集索引中的输入行。 如果存在 SET:() 谓词，则将每个更新的列设置为该值。 **Nonclustered Index Update** 是一个物理运算符。 |
| Online Index Insert          | **Online Index Insert** 物理运算符指示索引创建、更改或删除操作是在线执行的。 也就是说，基础表数据在索引操作期间仍然对用户可用。 |
| Parallelism                  | Parallelism 运算符（或交换迭代器）执行分发流、收集流和对流重新分区逻辑操作。 **Argument** 列可以包含一个 PARTITION COLUMNS:() 谓词和一个以逗号分隔的分区列的列表。 **Argument** 列还可以包含一个 ORDER BY:() 谓词，以列出分区过程中要保留排序顺序的列。 **Parallelism** 是物理运算符。 有关 Parallelism 运算符的详细信息，请参阅 [Craig Freedman 的博客系列](https://learn.microsoft.com/zh-cn/archive/blogs/craigfr/the-parallelism-operator-aka-exchange)。  **注意：** 如果查询已编译为并行查询，但在运行时作为串行查询运行，则通过 SET STATISTICS XML 或使用 SQL Server Management Studio 中的“包括实际执行计划”选项生成的显示计划输出将不会包含 Parallelism 运算符的 RunTimeInformation 元素 。 在 SET STATISTICS PROFILE 输出中，为 **Parallelism** 运算符显示的实际行计数和实际执行数将为零。 出现任何一种情况时，都意味着 **Parallelism** 运算符只在查询编译时使用，未在运行时查询计划中使用。 请注意，如果服务器上的并发负荷很高，则并行查询计划有时会以串行方式运行。 |
| Parameter Table Scan         | **Parameter Table Scan** 运算符扫描在当前查询中用作参数的表。 该运算符一般用于存储过程内的 INSERT 查询。 **Parameter Table Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Partial Aggregate            | **Partial Aggregate** 用于并行计划中。 它将聚合功能应用到尽可能多的输入行中，以便不必执行向磁盘写入数据的操作（称为“溢出”）。 Hash Match 是实现部分聚合的唯一一个物理运算符（迭代器）。 **Partial Aggregate** 是一个逻辑运算符。 |
| Population Query             | **Population Query** 运算符在打开游标时填充游标的工作表。    |
| Refresh Query                | **Refresh Query** 运算符为提取缓冲区中的行提取当前数据。     |
| Remote Delete                | **Remote Delete** 运算符用于从远程对象中删除输入行。 **Remote Delete** 既是一个逻辑运算符，也是一个物理运算符。 |
| Remote Index Scan            | **Remote Index Scan** 运算符可以扫描在 Argument 列中指定的远程索引。 **Remote Index Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Remote Index Seek            | **Remote Index Seek** 运算符利用远程索引对象的查找功能来检索行。 **Argument** 列包含所使用的远程索引名称和 SEEK:() 谓词。 **Remote Index Seek** 是一个逻辑物理运算符。 |
| Remote Insert                | **Remote Insert** 运算符将输入行插入到远程对象。 **Remote Insert** 既是一个逻辑运算符，也是一个物理运算符。 |
| Remote Query                 | **Remote Query** 运算符将查询提交给远程源。 发送给远程服务器的查询文本显示在 **Argument** 列中。 **Remote Query** 既是一个逻辑运算符，也是一个物理运算符。 |
| Remote Scan                  | **Remote Scan** 运算符扫描远程对象。 远程对象的名称显示在 **Argument** 列中。 **Remote Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Remote Update                | **Remote Update** 运算符将更新远程对象中的输入行。 **Remote Update** 既是一个逻辑运算符，也是一个物理运算符。 |
| Repartition Streams          | Repartition Streams 运算符（或交换迭代器）使用多个流并生成多个记录流。 记录的内容和格式不会改变。 如果查询优化器使用位图筛选器，则输出流中行的数量将减少。 输入流中的每个记录都放入一个输出流中。 如果该运算符保留次序，则必须对所有输入流排序并将它们合并到几个有序的输出流中。 如果将输出分区，那么 **Argument** 列会包含 PARTITION COLUMNS:() 谓词和分区依据列。如果输出已经排序，则 **Argument** 列包含一个 ORDER BY:() 谓词和已经排序的列。 **Repartition Streams** 是一个逻辑运算符。 该运算符只用于并行查询计划中。 |
| 结果                         | **Result** 运算符是查询计划结束时返回的数据。 它通常是显示计划的根元素。 **Result** 是一个语言元素。 |
| RID Lookup                   | **RID Lookup** 是使用提供的行标识符 (RID) 在堆上进行的书签查找。 **Argument** 列包含用于查找表中的行的书签标签和从中查找行的表的名称。 **RID Lookup** 通常带有 NESTED LOOP JOIN。 **RID Lookup** 是一个物理运算符。 有关书签查找的详细信息，请参阅 MSDN SQL Server 博客中的[Bookmark Lookup](https://learn.microsoft.com/zh-cn/archive/blogs/craigfr/)（书签查找）。 |
| Row Count Spool              | **Row Count Spool** 运算符扫描输入，计算现有的行数并返回相同数目的不包含任何数据的行。 必须检查现有行数（而非行中包含的数据）时，使用此运算符。 例如，如果 **Nested Loops** 运算符执行左半联接操作且联接谓词应用于内部输入，则可以在 **Nested Loops** 运算符内部输入的顶部放置行计数假脱机。 这样， **Nested Loops** 运算符就可以确定行计数假脱机输出的行数（因为不需要内侧的实际数据）以决定是否返回外部行。 **Row Count Spool** 是一个物理运算符。 |
| Right Anti Semi Join         | 当第一个（顶端）输入中不存在匹配行时， **Right Anti Semi Join** 运算符返回第二个（底端）输入中的每一行。 匹配行的定义是满足 **Argument** 列中的谓词的行（如果不存在谓词，则每行都是一个匹配行）。 **Right Anti Semi Join** 是一个逻辑运算符。 |
| Right Outer Join             | **Right Outer Join** 运算符返回满足联接第二个（底端）输入与第一个（顶端）输入中每个匹配行的每一行。 此外，它还返回第二个输入中在第一个输入中没有匹配行的任何行，即与 NULL 联接。 如果 **Argument** 列内不存在任何联接谓词，则每行都是一个匹配行。 **Right Outer Join** 是一个逻辑运算符。 |
| Right Semi Join              | 当第一个（顶端）输入中存在匹配行时， **Right Semi Join** 运算符返回第二个（底端）输入中的每一行。 如果 **Argument** 列内不存在任何联接谓词，则每行都是一个匹配行。 **Right Semi Join** 是一个逻辑运算符。 |
| 段                           | **Segment** 既是一个物理运算符，也是一个逻辑运算符。 它基于一个或多个列的值将输入集划分成多个段。 这些列显示为 **Segment** 运算符中的参数。 然后此运算符每次输出一个段。 |
| 序列                         | **Sequence** 运算符驱动大范围的更新计划。 就其功能而言，该运算符按顺序（从上到下）执行每个输入。 每个输入通常是不同对象的更新。 该运算符只返回其上一个（底端）输入中的行。 **Sequence** 既是一个逻辑运算符，也是一个物理运算符。 |
| Sequence Project             | **Sequence Project** 运算符将添加列以便计算有序集。 它基于一个或多个列的值将输入集划分成多个段。 然后此运算符每次输出一个段。 这些列在 **Sequence Project** 运算符中作为参数显示。 **Sequence Project** 既是一个逻辑运算符，也是一个物理运算符。 |
| Segment Repartition          | 在并行查询计划中，有时存在迭代器的概念性区域。 此类区域中的所有迭代器都可通过并行线程执行。 这些区域本身必须串行执行。 单个区域内的某些 **Parallelism** 迭代器称为 **Branch Repartition**。 两个这样的区域边界上的 **Parallelism** 迭代器称为 **Segment Repartition**。 **Branch Repartition** 和 **Segment Repartition** 是逻辑运算符。 |
| 快照                         | **Snapshot** 运算符创建一个看不到其他人所做更改的游标。      |
| Sort                         | **Sort** 运算符可对所有传入的行进行排序。 **Argument** 列包含 DISTINCT ORDER BY:() 谓词（如果此操作删除了重复项），或 ORDER BY:() 谓词（如果对以逗号分隔的列列表进行排序）。 如果按升序对列排序，则使用值 ASC 作为列的前缀；如果按降序对列排序，则使用值 DESC 作为列的前缀。 **Sort** 既是一个逻辑运算符，也是一个物理运算符。 |
| Split                        | **Split** 运算符用于优化更新处理。 它将每个更新操作拆分成删除和插入操作。 **Split** 既是一个逻辑运算符，也是一个物理运算符。 |
| Eager Spool                  | **Eager Spool** 运算符获取整个输入，并将每行存储在 **tempdb** 数据库中存储的隐藏临时对象中。 如果重绕该运算符（例如通过 **Nested Loops** 运算符重绕），但不需要任何重新绑定，则将使用假脱机数据，而不用重新扫描输入。 如果需要重新绑定，则将放弃假脱机数据，并通过重新扫描（重新绑定的）输入重新生成假脱机对象。 **Eager Spool** 运算符按“急切”方式生成自己的假脱机文件：当假脱机的父运算符请求第一行时，假脱机运算符将获取所有来自其输入运算符的行并将其存储在假脱机中。 **Eager Spool** 是一个逻辑运算符。 |
| Lazy Spool                   | **Lazy Spool** 逻辑运算符将其输入中的每一行存储到 **tempdb** 数据库内存储的隐藏临时对象中。 如果重绕该运算符（例如通过 **Nested Loops** 运算符重绕），但不需要任何重新绑定，则将使用假脱机数据，而不用重新扫描输入。 如果需要重新绑定，则将放弃假脱机数据，并通过重新扫描（重新绑定的）输入重新生成假脱机对象。 **Lazy Spool** 运算符以“迟缓”方式生成其假脱机文件，即每当假脱机父运算符请求一行时，假脱机运算符便从其输入运算符获取一行，然后将该行存储在假脱机中，而不是一次处理所有行。 Lazy Spool 是一个逻辑运算符。 |
| Spool                        | **Spool** 运算符将中间查询结果保存到 **tempdb** 数据库中。   |
| Stream Aggregate             | **Stream Aggregate** 运算符按一列或多列对行分组，然后计算由查询返回的一个或多个聚合表达式。 此运算符的输出可供查询中的后续运算符引用和/或返回到客户端。 **Stream Aggregate** 运算符要求输入在组中按列进行排序。 如果由于前面的 **Sort** 运算符或已排序的索引查找或扫描导致数据尚未排序，则优化器将在此运算符前面使用一个 **Sort** 运算符。 在 SHOWPLAN_ALL 语句或 SQL Server Management Studio中的图形执行计划中，GROUP BY 谓词中的列会列在 **Argument** 列中，而聚合表达式则列在 **Defined Values** 列中。 **Stream Aggregate** 是一个物理运算符。 |
| 开关                         | **Switch** 是一种特殊类型的串联迭代器，它具有 *n* 个输入。 有一个表达式与每个 **Switch** 运算符关联。 根据表达式的返回值（在 0 到 *n*-1 之间）， **Switch** 将适当的输入流复制到输出流中。 **Switch** 的一种用途是与某些运算符（如 **TOP** 运算符）一起实现涉及快进游标的查询计划。 **Switch** 既是一个逻辑运算符，也是一个物理运算符。 |
| Table Delete                 | **Table Delete** 物理运算符删除查询执行计划的 **Argument** 列中所指定表中的行。 |
| Table Insert                 | **Table Insert** 运算符将输入的行插入到在查询执行计划的 **Argument** 列指定的表中。 **Argument** 列还包含一个 SET:() 谓词，用于指示为每一列设置的值。 如果 **Table Insert** 的插入值没有子项，插入的行则来自 Insert 运算符本身。 **Table Insert** 是一个物理运算符。 |
| Table Merge                  | **Table Merge** 运算符可将合并数据流应用到堆。 该运算符可在其 **Argument** 列中所指定的表中删除、更新或插入行。 执行的实际操作取决于该运算符的 **Argument** 列中指定的 **ACTION** 列的运行时值。 **Table Merge** 是一个物理运算符。 |
| Table Scan                   | **Table Scan** 运算符从查询执行计划的 **Argument** 列所指定的表中检索所有行。 如果 WHERE:() 谓词出现在 **Argument** 列中，则仅返回满足此谓词的那些行。 **Table Scan** 既是一个逻辑运算符，也是一个物理运算符。 |
| Table Spool                  | **Table Spool** 运算符扫描输入，并将各行的一个副本放入隐藏的假脱机表中，此表存储在 [tempdb](https://learn.microsoft.com/zh-cn/sql/relational-databases/databases/tempdb-database?view=sql-server-ver16) 数据库中并且仅在查询的生存期内存在。 如果重绕该运算符（例如通过 **Nested Loops** 运算符重绕），但不需要任何重新绑定，则将使用假脱机数据，而不用重新扫描输入。 **Table Spool** 是一个物理运算符。 |
| Window Spool                 | **Window Spool** 运算符将每个行扩展为表示与行关联的窗口的行集。 在查询中，OVER 子句定义查询结果集内的窗口和窗口函数，然后计算窗口中的每个行的值。 **Window Spool** 既是一个逻辑运算符，也是一个物理运算符。 |
| Table Update                 | **Table Update** 物理运算符更新查询执行计划的 **Argument** 列中所指定表中的输入行。 SET:() 谓词确定每个更新列的值。 可以在 SET 子句中、此运算符内的其他位置以及此查询内的其他位置引用这些值。 |
| Table-valued Function        | **Table-valued Function** 运算符 (Transact-SQL 或 CLR) 计算表值函数，并将生成的行存储在 [tempdb](https://learn.microsoft.com/zh-cn/sql/relational-databases/databases/tempdb-database?view=sql-server-ver16) 数据库中。 当父迭代器请求这些行时， **Table-valued Function** 将返回 **tempdb**中的行。  调用表值函数的查询生成具有 **Table-valued Function** 迭代器的查询计划。 可以使用不同的参数值计算**Table-valued Function** ：  - **Table-valued Function XML Reader** 输入 XML BLOB 作为参数，并生成一个按 XML 文档顺序表示 XML 节点的行集。 其他输入参数可能会将返回的 XML 节点限于 XML 文档的子集。  -**Table Valued Function XML Reader with XPath filter** 是一种特殊类型的 **XML Reader Table-valued Function** ，它将输出限于满足 XPath 表达式的 XML 节点。  **Table-valued Function** 既是一个逻辑运算符，也是一个物理运算符。 |
| Top N Sort                   | **Top N Sort** 与 **Sort** 迭代器类似，差别仅在于前者需要前 *N* 行，而不是整个结果集。 如果 *N*的值较小， SQL Server 查询执行引擎将尝试在内存中执行整个排序操作。 如果 *N*的值较大，查询执行引擎将使用更通用的排序方法（该方法不采用 *N* 作为参数）重新排序。 |
| 顶部                         | **Top** 运算符扫描输入，但仅基于排序顺序返回最前面的指定行数或行百分比。 **Argument** 列可以包含要检查重复值的列的列表。 在更新计划中， **Top** 运算符用于强制实施行计数限制。 **Top** 既是一个逻辑运算符，也是一个物理运算符。 |
| UDX                          | 扩展运算符 (UDX) 可以实现 SQL Server中的一种 XQuery 或 XPath 操作。 所有 UDX 运算符既是逻辑运算符，又是物理运算符。  扩展运算符 (UDX) **FOR XML** 用于将其输入的关系行集序列化为 XML 表示形式，并以单个输出行、单个 BLOB 列的形式存储。 它是区分顺序的 XML 聚合运算符。  扩展运算符 (UDX) **XML SERIALIZER** 是区分顺序的一种 XML 聚合运算符。 它以 XML 文档顺序输入表示 XML 节点或 XQuery 标量的行，并在单个输出行、单个 XML 列中生成序列化的 XML BLOB。  扩展运算符 (UDX) **XML FRAGMENT SERIALIZER** 是一种特殊类型的 **XML SERIALIZER** ，用于处理表示在 XQuery 插入数据修改扩展中插入的 XML 片断的输入行。  扩展运算符 (UDX) **XQUERY STRING** 计算表示 XML 节点的输入行的 XQuery 字符串值。 它是一个区分顺序的字符串聚合运算符。 它输出一行多列，表示包含输入字符串值的 XQuery 标量。  扩展运算符 (UDX) **XQUERY LIST DECOMPOSER** 是一个 XQuery 列表分解运算符。 对于表示 XML 节点的每个输入行，它至少生成表示 XQuery 标量的一个行，如果输入的是 XSD 列表类型的行，则每个行都包含一个列表元素值。  扩展运算符 (UDX) **XQUERY DATA** 在表示 XML 节点的输入行上计算 XQuery fn:data() 函数的值。 它是一个区分顺序的字符串聚合运算符。 它输出一行多列，表示包含 **fn:data()** 结果的 XQuery 标量。  扩展运算符 **XQUERY CONTAINS** 在表示 XML 节点的输入行上计算 XQuery fn:contains() 函数的值。 它是一个区分顺序的字符串聚合运算符。 它输出一行多列，表示包含 **fn:contains()** 结果的 XQuery 标量。  扩展运算符 **UPDATE XML NODE** 更新 XML 类型上的 **modify()** 方法中 XQuery 替换数据修改扩展中的 XML 节点。 |
| Union                        | **Union** 运算符扫描多个输入，输出扫描的每一行并删除重复项。 **Union** 是一个逻辑运算符。 |
| Update                       | **Update** 运算符更新在查询执行计划的 **Argument** 列中所指定对象中的每一输入行。 **Update** 是一个逻辑运算符。 物理运算符为 **Table Update**、 **Index Update**或 **Clustered Index Update**。 |
| While                        | **While** 运算符实现 Transact-SQL while 循环。 **While** 是一个语言元素。 |