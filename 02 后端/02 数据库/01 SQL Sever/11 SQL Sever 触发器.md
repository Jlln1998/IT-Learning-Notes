#  SQL Sever 触发器

## 1. 定义

> 触发器为特殊类型的存储过程。
>
> 触发器可监听系统create、alter、insert、update、delete等T-SQL语句的执行，在执行时自动触发一些事件。

## 2. 触发器类型

> 触发器按触发时间，可分为
>
> * AFTER 触发器，又称事后触发器，即在监听的操作执行完之后再触发。
> * INSTEAD OF 触发器，又称替代触发器，即用触发器设定的操作替代监听到的事件操作。
>
> 触发器按作用对象，可分为
>
> * 表级触发器，针对某张表设置的触发器。（不常用）
> * 数据库触发器，针对某个数据库设置的触发器。

## 3. 使用触发器

### 3.1 创建和修改

````SQL
{CREATE|ALTER} TRIGGER 触发器名称 ON {表名|数据库名} -- 设定是表级、还是数据库级
{AFTER|INSTEAD OF}	-- 设定是事后触发器、还是替代触发器
{[INSERT][ , ][UPDATE] [ , ][DELETE]}	-- 要监听的操作，也可以同时监听
AS
BEGIN
	-- T-SQL语句
END
````

### 3.2 触发器开启和关闭

> 创建触发器之后，如果我们可以手动控制触发器的关闭和开启

````sql
-- 关闭某个触发器
DISABLE TRIGGER 触发器名称 ON {表名|DATABASE}

-- 开启某个触发器
ENABLE TRIGGER 触发器名称 ON {表名|DATABASE}

-- 开启和关闭所有触发器
ENABLE TRIGGER ALL ON {表名|DATABASE}

-- 查看触发器是开启还是关闭的,查看下表的 is_disabled字段，是0表示开启。
SELECT TOP (100) * FROM sys.triggers where name='触发器名'
````

### 3.3 删除触发器

> 删除之后无法恢复

````sql
DROP TRIGGER 触发器名称
````

### 3.4 触发器中的临时表

> 当我们创建了作用于表对象的触发器，那么在触发器被触发时，系统会在内存中创建两张只读的、且表结构和触发器作用的表一模一样的临时表，表中存放我们的改动记录。
>
> 这两张表分别是 inserted 表和 deleted 表，表中存放数据会根据触发器监听的操作而变动，情况如下：

| 操作   | inserted表       | deleted表        |
| ------ | ---------------- | ---------------- |
| insert | 存放新增的记录   | —                |
| delete | —                | 存放被删除的记录 |
| update | 存放修改后的记录 | 存放修改前的记录 |

## 4. 案例

### 4.1 触发器做版本记录

> 利用触发器将每次存储过程的修改代码记录到版本表

* 创建一个表，用于存储每次修改的代码，以及修改人、修改的DB等一些信息。

````sql
CREATE TABLE dbo.ProcedureVersions
(
    EventDate    DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    EventType    NVARCHAR(100),
    EventDDL     NVARCHAR(MAX),
    DatabaseName NVARCHAR(255),
    SchemaName   NVARCHAR(255),
    ObjectName   NVARCHAR(255),
    HostName     NVARCHAR(255),
    IPAddress    VARCHAR(32),
    ProgramName  NVARCHAR(255),
    LoginName    NVARCHAR(255)
);
GO
````

* 创建数据库级别的触发器，监听存储过程的 Create、Alter、Drop

> EVENTDATA() 函数，可获取数据库执行事件，格式XML格式。

````sql
CREATE TRIGGER [CaptureStoredProcedureChanges]
    ON DATABASE
    FOR CREATE_PROCEDURE, ALTER_PROCEDURE, DROP_PROCEDURE -- 注意下划线格式
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @EventData XML = EVENTDATA(), 
    		@ip VARCHAR(32);

    SELECT @ip = client_net_address
    FROM sys.dm_exec_connections
    WHERE session_id = @@SPID;

    INSERT dbo.ProcedureChanges
    (
        EventType,
        EventDDL,
        SchemaName,
        ObjectName,
        DatabaseName,
        HostName,
        IPAddress,
        ProgramName,
        LoginName
    )
    SELECT
        @EventData.value('(/EVENT_INSTANCE/EventType)[1]',   'NVARCHAR(100)'), 
        @EventData.value('(/EVENT_INSTANCE/TSQLCommand)[1]', 'NVARCHAR(MAX)'),
        @EventData.value('(/EVENT_INSTANCE/SchemaName)[1]',  'NVARCHAR(255)'), 
        @EventData.value('(/EVENT_INSTANCE/ObjectName)[1]',  'NVARCHAR(255)'),
        DB_NAME(),
        HOST_NAME(),
        @ip, 
        PROGRAM_NAME(),
        SUSER_SNAME();
END
GO
````

