# Oracle DB笔记

## Oracle 体系结构
Oracle 系统体系结构由三部分组成：

1. 逻辑结构
2. 物理结构
3. 实例：是维系物理结构和逻辑结构的核心

## 逻辑结构
Oracle 逻辑结构是一种**层次结构**。主要由：表空间、段、区和数据块等概念组成。逻辑结构是面向用户的，用户使用 Oracle 开发应用程序使用的就是逻辑结构。数据库存储层次结构及其构成关系，结构对象从数据块到表空间形成了不同层次的粒度关系。

### 数据块
Oracle 数据块（Data Block）是一组连续的操作系统块。分配数据块大小是在 Oracle 数据库创建时设置的，数据块是 Oracle 读写的基本单位。

数据块的结构主要包括：

|名称|简介|
|:---:|:---:|
|标题|包括一般的块信息，如块地址，段类型等|
|表目录|包括有关表在该数据块中的行信息|
|行目录|包括有关在该数据块中行地址等信息|
|行数据|包括表或索引数据。一行可跨越多个数据块|
|空闲空间|空闲空间是用于插入新的行和需要额外空间的行更新|

在数据操作中，有两种语句可以增加数据库块的空闲空间：一个是`Delete`删除语句，另一个是`Update`更新现有行。

能够对空闲空间产生影响的参数有两个：`pctfree`和`pctused`。对于手工管理的表空间，在特定段中的所有数据块，可使用两个空间管理参数`pctfree`和`pctused`来控制`insert`和`update`对空闲空间的使用。当创建或修改表时可指定这两个参数。创建或修改一个拥有自己的索引段的索引时可指定`pctfree`参数。

### 区
区（Extent）也称为数据区，是一组连续的数据块。当一个表、回滚段或临时段创建或需要附加空间时，系统总是为之分配一个新的数据区。**一个数据区不能跨越多个文件**，因为它包含连续的数据块。使用区的目的是用来保存特定数据类型的数据，也是表中**数据增长的基本单位**。在 Oracle 数据库中，分配空间就是以数据区为单位的。一个 Oracle 对象包含至少一个数据区。

### 段
段（Segment）是由多个数据区构成的，它是为特定的数据库对象（如表段、索引段、回滚段、临时段）分配的一系列数据区。段内包含的数据区可以不连续，并且可以跨越多个文件。使用段的目的是用来保存特定对象。

### 表空间
Oracle 数据库是由若干个表空间（Table space）构成的。任何数据库对象在存储时都必须存储在某个表空间中。表空间对应于若干个磁盘文件，即表空间是由一个或多个磁盘文件构成的。表空间相当于操作系统中的文件夹，也是数据库逻辑结构与物理文件之间的一个映射。每个数据库至少有一个表空间，表空间的大小等于所有从属于它的数据文件大小的总和。

## 物理结构
Oracle 数据库主要由三种文件构成：

1. 数据文件
2. 控制文件
3. 重做日志文件

### 数据文件
表空间最终都是以若干个数据文件的形式存储的。数据文件中主要包括：表中的数据、索引数据、数据字典定义、数据库对象、用户定义、排序时产生的临时数据以及为保证事务重做所必需的数据。

由于数据对象类型不同，因此按照不同的表空间分类存储，这样可以高效提高系统的整体性能。

### 控制文件
控制文件是一个**二进制**文件，用来描述数据库的物理结构，一个数据库只需要一个控制文件，控制文件的内容包括：

- 数据库名及数据库唯一标识
- 数据库创建时间
- 表空间名字
- 数据文件和日志文件名字及位置
- 数据库恢复所需的同步信息，即检查点号

如果控制文件损坏或丢失，数据库将终止并且无法启动，所以，要对控制文件进行镜像，手动镜像步骤如下：

1. 关闭数据库
2. 复制控制文件
3. 修改参数文件，加入新增的控制文件位置描述
4. 重新启动数据库

### 重做日志文件
在 Oracle 中，日志文件也叫作重做日志文件或重演日志文件（Redo Log Files）。日志文件用于记录对数据库的修改信息，对数据库所做的修改信息都被记录在日志中。这包括用户对数据库中数据的修改和数据库管理员对数据库结构的修改。如果，只是对数据库中的信息进行查询操作，则不会产生日志信息。

在 Oracle 数据库中，日志文件是**成组使用**的。日志文件的组织单位叫日志文件组，日志文件组中的日志文件叫日志成员。每个 Oracle 数据库系统都有多个日志文件组，每一组有一个或多个日志成员（即多个日志文件组成）。

## 参数文件
除了构成 Oracle 数据库物理结构的三类主要文件外，Oracle 数据库还具有另外一种重要的文件：参数文件。

参数文件记录了 Oracle 数据库的基本参数信息，主要包括数据库名、控制文件所在路径、进程等。

其中有两个重要的参数文件：pfile 和 spfile

这两个文件中所存放的是在 Oracle 创建特定数据库过程中以及以后启动和关闭数据库实例时所用到的有关控制文件和数据库的参数信息。

### pfile
pfile 参数文件的具体内容包括：数据库作业队列的最大数、实例标识、安全与审计、数据库标识、数据块大小、数据文件最大数、控制文件路径、内存结构，可以编辑。

### spfile
spfile 是用于服务器端且在数据库启动时默认使用的二进制参数文件，是不可编辑的。

### pfile 和 spfile
它们都是数据库启动必不可少的参数，spfile 是服务器端参数文件。开始创建数据库时，不论是手工创建还是使用向导创建，都必须使用 pfile 参数。一旦数据库创建完成后，必须通过 pfile 来创建 spfile。这样，Oracle 以自动方式启动数据库时，才能直接使用服务器端 spfile 参数文件，而不必指定参数文件的位置。

## SQL *PLUS 语言
### SQL的特点
1. 非过程化语言

2. 用户性能好

3. 提供有“视图”数据结构

	SQL 可以对两种基本的数据结构进行操作，**表**与**视图**
	
	**表**是 Oracle 数据库中存放数据实体的基本逻辑存储单位（称为基表）。
	
	**视图**是在基表上所建的**逻辑表**，可由某一基表的某些列或某些行的数据项组成，也可由多个基表满足一定条件的数据项组成。**视图是一个窗口**，通过它可以观察和改变表中信息，但不包含真正存储的数据，不占存储空间（故称虚表），用户对视图操作如同对表操作一样。

4. 两种使用方式

	- 交互式命令方式
	- 嵌入方式（程序方式）
	
5. 语言功能强

	按 SQL 提供的命令实现的功能划分，通常分为四类：
	
	1. 数据查询语言（DQL，Data Query Language）
	
		用于对数据库中数据的各种查询。其基本结构是由`select from`和`where`子句组成的查询语句
	
	2. 数据定义语言（DDL，Data Definition Language）
	
		用于数据库数据结构的定义，并在数据字典中生成数据项
		
		常用的语句有：`create`、`revoke`、`grant`、`alter`
	
	3. 数据操作语言（DML，Data Manipulation Language）
	
		用于数据库中数据的增删改查
		
		常用的语句有：`delete`、`insert`、`update`
	
	4. 数据控制语言（DCL，Data Control Language）
	
		用于数据的保存、恢复和约束
		
		常用的语句有：`commit`、`rollback`、主/外码定义
		
### SQL *PLUS
SQL *PLUS 是 Oracle 系统中功能强大的第四代语言工具，它基于 SQL 但又具有 Oracle 特定功能，是 Oracle 提供的一个用于处理 Oracle 数据和生成报表等功能的一种语言工具。

它包括两部分内容：

1. 标准 SQL 命令：

	主要对数据库中数据操作
	
2. SQL *PLUS 命令：

	主要用来完成编辑，保存，执行 SQL 命令，控制查询结果的显示方式等功能
	
### SQL *PLUS 命令汇集

|SQL *PLUS编辑命令|定义|格式|
|:---:|:---:|:---:|
|APPEND/A|在行末增加文件||
|CHANGE/old/new/C|更改行||
|CLEAR BUFFER/CL|删除缓冲区中所有行||
|DEL|删除当前行||
|INPUT/I|添加一行或多行||
|LIST/L|列出 SQL 缓冲区中所有行|L [行数]|
|RUN/'/'|执行 SQL 缓冲区中语句|RUN [文件名]|
|EDIT|调用操作系统的编辑器|EDIT [文件名]|
|SAVE|保存缓冲区中内容|SAVE <文件名>|
|GET|将文件中的内容添加到 SQL 缓冲区中|GET <文件名>|
|START/'@'|执行含有 SQL 命令和 SQL *PLUS命令组成的成批命令文件|START <文件名>|
|'$'/'!'|执行操作系统命令行，例如：SQL>$dir|$<命令>|
|SHOW|用来显示 SQL *PLUS 系统变量|SHOW <变量名>|
|SET|用来改变指定参数（系统变量）的值|SET <选项> <值或开关状态>|
|SPOOL|查询结果输出命令|SPOOL <数据文件名>|
|TTITLE|头标题输出命令|TTITLE <选项> <正文>|
|BTITLE|尾标题输出命令|BTITLE <选项> <正文>|
|COLUMN|指定列标题及数据格式|COLUMN <列名> [LIKE <表中另一列名>] FORMAT 格式 HEADING <栏名>|
|BREAK|对查询结果进行分组|BREAK ON <列名1> [ON <列名2> ...] [SKIP N]|
|COMPUTE|进行分组统计计算|COMPUTE <函数> OF <列名1> ON <列名2>|

### SQL语言查询功能
查询功能主要由`select`语句实现，其基本形式：

```sql
select <列名表或*> 
from <表名/视图名>
[where <条件>]
[group by <分组内容>]
[having <组内条件>]
[order by <排序方式>]
```

使用`distinct`修饰查找，只显示不同的行，相同重复的行被删去

使用`order by`修饰查找，按指定规则进行排序

#### 查询命令中的表达式
SQL 语言中，可使用**算数**、**关系**、**逻辑**运算符组成的表达式作为条件，各中表达式中运算符及优先级：

1. 算术符：

	`+`、`-`、`*`、`/`、`||`

	先`*`、`/`，后`+`、`-`、`||`
	
2. 关系符：

	`=`、`!=`或`<>`、`>`、`>=`、`<`、`<=`
	
	`IN`等价于`=ANY`
	
	`NOT IN`等价于`!=ALL`
	
	`BETWEEN...AND`在两值之间
	
	`LIKE`使用字符型匹配
	
	`IS NULL`、`IS NOT NULL`（NULL不同于数值0，也不同于空格符）
	
3. 逻辑符：

	`NOT`、`AND`、`OR`
	
关系、逻辑表达式一般用在`where`子句中和`having`子句中

#### 查询命令中的集函数

|名称|作用|
|:---:|:---:|
|count|统计数量|
|avg|平均值|
|min|最小值|
|max|最大值|
|sum|累加和|

#### 分组查询
分组查询就是将数据库表中的数据按给定条件分类组合，然后再根据需要分别操作。

利用命令中的`group by`子句，可根据某字段值分为多组，然后操作，从而控制和影响查询结果。

#### 多表连接查询
在关系数据库中，各相关表之间的连接关系是通过**字段（共有字段）**值来体现的。

#### 子查询（嵌套查询）
在一个`select`命令中的`where`子句中，若又出现一个`select`命令，该查询称为**嵌套查询**，`where`子句中包括的查询称为**子查询（或嵌入的查询块）**，响应的外层的`select`命令为**主查询**。

### SQL 语言数据操作功能
#### 数据插入命令
1. 插入命令一般形式

	```sql
	insert into <表名> [(列名表)] values (<值表>);
	```
	
2. 带参数插入语句形式

	```sql
	insert into <表名> values (<参数表>);
	```
	
3. 带有子查询的插入语句（查询插入）

	```sql
	insert into <表名> [(列名表)] <select 语句>;
	```
	
#### 数据修改命令
一般形式：

```sql
update <表名>
set <列名1>=<表达式1> [, <列名2>=<表达式2> ...]
[where <条件>];
```

#### 数据删除命令
一般形式：

```sql
delete from <表名>
[where <条件>];
```

### SQL 语言数据定义功能
#### 创建表结构命令
一般形式：

```sql
create table <表名>
(<字段1> <类型>
[, <字段2> <类型>, ...]);
```

#### 复制表结构命令
一般形式：

```sql
create table <表名>
as <select 语句>;
```

#### 修改表结构命令
1. 增加新列名命令：

```sql
alter table <表名> add (<列定义>);
```

2. 修改列定义命令：

```sql
alter table <表名> modify (<列定义>);
```

#### 删除表命令
一般形式：

```sql
drop table <表名>
```

#### 对表重新命令
一般形式：

```sql
rename <原表名> to <新表名>
```

### 有关视图的操作
视图时一种特殊的表，是建立在基表上的虚表，是基表的一个数据窗口，通过视图可对表中数据操作，是实现对数据的保密及数据的安全性的一种手段。

#### 创建视图
一般格式：

```sql
create view <视图名> [(<视图列名表>)]
as <select 语句>
[with check option];
```

#### 操作视图
通过视图可对基表中数据进行增删改查等操作，对用户来讲，对视图的操作如同对表一样。

### 有关索引的操作
索引的操作包含建立索引和删除索引
建立索引的目的用于查询：

1. 为了提高查询速度

2. 为保证表中某一列值的唯一性

3. 为能加速连接查询

#### 建立索引的命令
一般格式：

```sql
create [unique] index <索引名>
on <表名> (<列名1> [ASC/DESC]
[, <列名2> [ASC/DESC]]...);
```

#### 删除索引的命令
一般格式：

```sql
drop index <索引名>
[on <表名>];
```

#### 建立索引的准则：
1. 表中行数为几百以上，表越大，使用索引越能提高检索效率，索引越多，数据库就有较多机会找捷径，为主要用于查询的大表建索引

2. 尽量不要再一表上建立多个索引。因索引需要磁盘空间，同时会降低数据更新的执行速度，因库中数据更新，涉及到建索引列值的改变，系统会维护索引，索引越多维护工作量越大。所以建索引能提高检索速度，但同时降低了更新速度

3. 为经常在查询中、表连接中用到的列建索引

### 创建同义词
同义词是数据库对象。就是用户为表、视图或其他同义词建立一个别名

若为表建立了同义词，就可以通过同义词对该表进行操作

其优点：

1. 隐藏表的拥有者或表名

2. 隐藏表的具体位置

3. 使用简单方便

#### 建立同义词命令
一般格式：

```sql
create [public] synonym
<同义词名> for <[用户名]表名>;
```

选项`public`是 DBA 有权限创建公共同义词，供全体用户使用，无选项`public`，则为私用同义词，只供本用户自己用或授过权的用户使用

#### 删除同义词命令
一般格式：

```sql
drop [public] synonym <同义词名>
```