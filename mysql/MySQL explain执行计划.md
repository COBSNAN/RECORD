---
title: MySQL explain执行计划
tags: mysql
grammar_cjkRuby: true
---
![enter description here][1]

- 如上图，其中`describe`或`explain` 一个表名的时候，会展示这个表的字段信息
- 通过`explain`查询语句来查看这个语句的执行计划

> 执行计划中各字段的含义

- id：select查询的序列号，id相同，执行顺序由上至下,在所有组中，id值越大，优先级越高，越先执行
- **select_type**：select查询的类型，主要是区别普通查询和联合查询、子查询之类的复杂查询
	- SIMPLE：查询中不包含子查询或者UNION
	- PRIMARY：查询中若包含任何复杂的子部分，最外层查询被标记为PRIMARY
	- SUBQUERY：在SELECT或WHERE列表中包含子查询，该子查询被标记为SUBQUERY
	- DERIVED：在FROM列表中包含的子查询被标记为DERIVED（衍生）
	- UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在 FROM子句的子查询中，外层SELECT将被标记为DERIVED
	- UNION RESULT：从UNION表获取结果的SELECT被标记为UNION RESULT

- table：输出的行所引用的表
- PARTITIONS：对于分区表，显示查询的分区ID，对于非分区表，显示为NULL
- **type** :联合查询所使用的类型，表示MySQL在表中找到所需行的方式，又称“访问类型”

性能 | 值 | 含义
-- | -- | --
高 | NULL | MySQL在优化过程中分解语句，执行时甚至不用访问表或索引
↓ | const, system | 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量。system是const类型的特例，当查询的表只有一行的情况下， 使用system
↓ | eq_ref | 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
↓ | ref | 非唯一性索引扫描，返回匹配某个单独值的所有行。常见于使用非唯一索引即唯一索引的非唯一前缀进行的查找
↓ | range | 扫描部分索引，索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行，常见于between、<、>等的查询
↓ | index | 扫描全部索引树
低 | ALL | 全表扫描

- possible_keys：指出MySQL能使用哪个索引在该表中找到行。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用。如果是空的，没有相关的索引。这时要提高性能，可通过检验WHERE子句，看是否引用某些字段，或者检查字段不是适合索引。 
- key：显示MySQL实际决定使用的键。如果没有索引被选择，键是NULL
- key_len：显示MySQL决定使用的键长度。表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。如果键是NULL，长度就是NULL。文档提示特别注意这个值可以得出一个多重主键里mysql实际使用了哪一部分。*key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的*
- ref：显示哪个字段或常数与key一起被使用
- rows：这个数表示mysql要遍历多少数据才能找到，表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数，在innodb上可能是不准确的。
- Filtered：表示返回结果的行数占需读取行数的百分比，Filter列的值越大越好
- Extra：额外信息

类型 | 含义
-- | --
Only index | 信息只用索引树中的信息检索出的，这比扫描整个表要快
using where | 使用上了where限制，表示MySQL服务器在存储引擎受到记录后进行“后过滤”（Post-filter），如果查询未能使用索引，Using where的作用只是提醒我们MySQL将用where子句来过滤结果集。
impossible where | 表示用不着where，一般就是没查出来啥
Using filesort | （MySQL中无法利用索引完成的排序操作称为“文件排序”）当我们试图对一个没有索引的字段进行排序时，就是filesoft。它跟文件没有任何关系，实际上是内部的一个快速排序。 
Using temporary | （表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询），使用filesort和temporary的话会很吃力，WHERE和ORDER BY的索引经常无法兼顾，如果按照WHERE来确定索引，那么在ORDER BY时，就必然会引起Using filesort，这就要看是先过滤再排序划算，还是先排序再过滤划算。
Distinct |优化distinct操作，在找到第一个匹配元组后既停止找同样值得动作
Not exists | 使用not exists 来优化查询

### 例子

> 全表扫描 由于name没有索引所以走的是全表扫描，注意Extra里的含义

![全表扫描][2]

> 扫描全部索引树

![扫描全部索引树][3]

> range扫描部分索引

![range扫描部分索引][4]

> ref 非唯一性索引扫描 和const

![ref 非唯一性索引扫描][5]

> eq_ref 唯一性索引扫描
![eq_ref 唯一性索引扫描][6]

### 关于MySQL执行计划的局限性

1. EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
2. EXPLAIN不考虑各种Cache
3. EXPLAIN不能显示MySQL在执行查询时所作的优化工作
4. 部分统计信息是估算的，并非精确值
5. EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看。

### 备注

> filesort是通过相应的排序算法,将取得的数据在内存中进行排序。

> MySQL需要将数据在内存中进行排序，所使用的内存区域也就是我们通过sort_buffer_size 系统变量所设置的排序区。这个排序区是每个Thread 独享的，所以说可能在同一时刻在MySQL 中可能存在多个 sort buffer 内存区域。


**在MySQL中filesort 的实现算法实际上是有两种**
- 双路排序：是首先根据相应的条件取出相应的排序字段和可以直接定位行数据的行指针信息，然后在sort buffer 中进行排序。
- 单路排序：是一次性取出满足条件行的所有字段，然后在sort buffer中进行排序


  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/1521672793513.jpg
  [2]: https://www.github.com/COBSNAN/ImageHub/raw/master/1521672924508.jpg
  [3]: https://www.github.com/COBSNAN/ImageHub/raw/master/1521673827114.jpg
  [4]: https://www.github.com/COBSNAN/ImageHub/raw/master/1521674024993.jpg
  [5]: https://www.github.com/COBSNAN/ImageHub/raw/master/1521674318465.jpg
  [6]: https://www.github.com/COBSNAN/ImageHub/raw/master/1521675720850.jpg