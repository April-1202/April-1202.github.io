# 一些mysql的优化点



这篇文章提供了mysql查询优化的一些方案和建议.

<!--more-->


## 一 前言

在使用sql进行查询的时候，就会经常会遇到查询速度过慢，因为掌握一些优化的小技巧，会大大帮助我们提高sql的查询速率

## 二 优化的思路和原则有哪些

1. 优化更需要优化的查询.
2. 从explain入手.
3. 用小结果集驱动大结果集.
4. 尽可能在索引中完成排序.
5. 只取出自己需要的字段.
6. 使用最有效的过滤条件.
7. 尽可能避免复杂的join.



## 三 合理利用Explain
explain关键字段的展示


字段     | 说明 
-------- | -----
select_type | 查询类型：<br>DEPENDENT SUBQUERY : 子查询中内层的第一个SELECT,依赖于外部查询结果集；<br>DEPENDENT UNION：子查询中的UNION中从第二个SELECT 开始的后面所有SELECT,同样依赖于外部查询结果集;<br>PRIMARY: 子查询中的最外层查询，不是主键查询；<br>SUBQUERY：子查询内层查询的第一个SELECT,结果不依赖于外部结果集；<br>UNCACHEABLE SUBQUERY：结果集无法缓存的子查询；<br>UNION：UNION语句中第二个SELECT开始的后面所有SELECT,第一个SELECT为PRIMARY<br>UNION RESULT：UNION中的合并结果 
table | 所访问数据库中表的名称 
type | 访问方式：<br>ALL: 全表扫描<br>const: 常量，最多只有一条记录匹配，由于是常量，所以实际上只需要读一次<br>eq_ref: 最多只有一条匹配结果，一般是主键或者唯一索引来访问的<br>index: 全索引扫描<br>range: 索引范围扫描<br>ref: jion语句中被驱动表索引的引用查询<br>system: 系统表，表中只有一行数据
possible_keys | 可能使用到的索引
key | 使用的索引
key_len | 索引长度
rows | 估算出来的结果集的总记录数
extra | 额外信息



## 四 合理利用索引

### 1 索引的利弊
优点：提高数据检索速度，降低数据库的io成本
缺点：查询需要更新索引信息带来额外的资源消耗，索引还会占用额外的存储空间

### 2 怎么样有效的建立索引
1. 较频繁的作为条件的查询字段应该创建索引
2. 更新频繁的字段不适合建立索引
3. 唯一性太差的字段不适合建立索引（比如：性别、状态）
4. 不出现在where中的字段不适合建立索引

## 五 一些实际sql语句中的优化实践

### 1. 应尽量避免在 where 子句中使用!=或<>操作符
否则引擎将放弃使用索引而进行全表扫描
### 2. 首先应考虑在 where 及 order by 涉及的列上建立索引
### 3. 应尽量避免在 where 子句中对字段进行 null 值判断
否则将导致引擎放弃使用索引而进行全表扫描

如：

• select id from t where num is null

可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：

• select id from t where num=0
### 4. 尽量避免在 where 子句中使用 or 来连接条件
否则将导致引擎放弃使用索引而进行全表扫描
如：

• select id from t where num=10 or num=20

可以这样查询：

• select id from t where num=10 
  union all
  select id from t where num=20
### 5. 模糊查询不能前置百分号
• 不走索引：select id from t where name like ‘%c%’

• 走索引：select id from t where name like ‘c%’
### 6. in 和 not in 也要慎用
如：

• select id from t where num in(1,2,3)

对于连续的数值，能用 between 就不要用 in 了：

• select id from t where num between 1 and 3

### 7. 如果在 where 子句中使用参数，也会导致全表扫描
因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。
如下面语句将进行全表扫描：

• select id from t where num=@num

可以改为强制查询使用索引：

• select id from t with(index(索引名)) where num=@num

### 8. 应尽量避免在 where 子句中对字段进行表达式操作
这将导致引擎放弃使用索引而进行全表扫描。如：

• select id from t where num/2=100

应改为:

• select id from t where num=100*2

### 9. 应尽量避免在where子句中对字段进行函数操作
如：

• select id from t where substring(name,1,3)=’abc’ –name以abc开头的id
select id from t where datediff(day,createdate,’2005-11-30′)=0 –’2005-11-30′生成的id

应改为:

• select id from t where name like ‘abc%’

select id from t where createdate>=’2005-11-30′ and createdate<’2005-12-1′

### 10. 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算
否则系统将可能无法正确使用索引

### 11. 使用联合索引时，遵循最做前缀原则

### 12. 很多时候用 exists 代替 in 是一个好的选择
• select num from a where num in(select num from b)

用下面的语句替换：

• select num from a where exists(select 1 from b where num=a.num)

### 13. 对于像性别这样有大量重复的字段建立索引没有效果

### 14. 索引并不是越多越好
索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数较好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要

### 15. 应尽可能的避免更新 clustered 索引数据列
因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引

### 16. 尽量使用数字型字段
若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了
 
### 17. 尽量不要使用 select * from t 
用具体的字段列表代替“*”，不要返回用不到的任何字段。



