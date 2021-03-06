## 《高性能 MySQL》 读书笔记四

### 第四章：Schema与数据类型优化

良好的逻辑设计和物理设计是高性能的基石，应该根据系统将要执行的查询语句来设计 schema，玩玩需要权衡各种因素。

#### 4.1 选择优化的数据类型

MySQL 支持的数据类型非常多，选择正确的数据类型对于获得高性能至关重要。

##### 更小的通常更好

​		一般情况下应该尽量使用可以正确存储数据的最小数据类型，但是要确保没有低估需要存储的值的范围。

##### 简单就好

​		简单数据类型的操作通常需要更少的CPU周期。例如：整型比字符操作代价更低、应该使用MySQL内建的类型而不是字符串存储日期和时间、应该用整型存储IP地址。

##### 尽量避免NULL

​		通常情况下最好指定列为 NOT NULL，除非真的需要存储 NULL 值。

​		如果查询中包含可为 NULL 的列，对 MySQL 来说更难优化，因为可为 NULL 的列使得索引、索引统计和值比较都更为复杂。可为 NULL 的列会使用更多的存储空间，在 MySQL 里也需要特殊处理。当可为 NULL 的列被索引时，每个索引记录需要一个额外的字节，在 MyISAM 里甚至还可能导致固定大小的索引 ( 例如只有一个整数列的索引 ) 变成可变大小的索引。

​		如果计划在列上建索引，就应该尽量避免设计成可为 NULL 的列。当然也有例外， 例如InnoDB 使用单独的位 (bit) 存储NULL值， 所以对于稀疏数据有很好的空间效率。

> ​		**在数据库中，稀疏数据是指在二维表中含有大量空值的数据；即稀疏数据是指，在数据集中绝大多数数值缺失或者为零的数据。稀疏数据绝对不是无用数据，只不过是信息不完全，通过适当的手段是可以挖掘出大量有用信息。**

在为列选择数据类型时，第一步需要确定合适的大类型：数字、字符串、时间等。

下一步是选择具体类型。例如：DATETIME 和 TIMESTAMP 列都可以存储相同类型的数据：时间和日期，精确到秒。然而 TIMESTAMP 只使用 DATETIME 一半的存储空间，并且会根据时区变化，但是TIMESTAMP允许的时间范围要小得多。

#### 4.1.1 整数类型

两种类型的数字：整数 ( whole number ) 和实数 ( real number )。存储整数可选：TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT。分别使用8，16，24，32，64位存储空间。它们可存储的值的范围从 $-2^{(N-1)}$ 到 $2^{(N-1)}-1$，其中 N 是存储空间的位数。

设置 UNSIGNED 属性表示不允许负值，大致可以使正数的上限提高一倍。

MySQL 可以为整数类型指定宽度，例如 INT(11)，对大多数应用是没有意义的。它不会限制值的合法范围，只是规定了MySQL的一些交互工具用来显示字符的个数。

#### 4.1.2 实数类型

实数是带有小数部分的数字，可以使用 DECIMAL 存储比 BIGINT 还大的整数。

DECIMAL 类型用于存储精确的小数。在MySQL 5.0 和更高版本，DECIMAL类型支持精度计算。FLOAT 和 DOUBLE 类型支持使用标准的浮点运算进行近似计算。

浮点类型在存储同样范围的值时，通常比 DECIMAL 使用更少的空间。FLOAT 使用4个字节存储，DOUBLE 占用8个字节。

MySQL 使用 DOUBLE 作为内部浮点计算的类型。

因为需要额外的空间和计算开销，所以应该尽量只在对小数进行精确计算时才使用 DECIMAL ——例如存储财务数据。在数据量较大时可以考虑使用 BIGINT 代替 DECIMAL，将需要存储的货币单位根据小数的位数乘以相应的倍数即可。

#### 4.1.3 字符串类型

##### VARCHAR 和 CHAR 类型

如何存储在磁盘和内存中跟存储引擎的具体实现有关。存储引擎存储 CHAR 或者 VARCHAR 值的方式在内存中和在磁盘上可能不一样。

###### VARCHAR

​		VARCHAR 类型用于存储可变长字符串，比定长类型更节省空间。但是如果MySQL表使用 ROW_FORMAT=FIXED 创建的话，每一行都会使用定长存储，会浪费空间。

​		VARCHAR 需要使用 1 或 2 个额外字节记录字符串的长度。

​		VARCHAR 节省了存储空间，所以对性能也有帮助。但由于变长，在 UPDATE 时可能使行变得比原来长，导致需要做额外的工作。比如一个行占用空间增长，并且页内没有更多的空间可以存储的情况下，不同存储引擎处理方式会不一样。

​		适合使用VARCHAR的情况：字符串列的最大长度比平均长度大很多、列的更新很少、使用了 UTF-8 这种复杂字符集，每个字符使用不同字节数存储。

###### CHAR

​		CHAR 类型是定长的：MySQL总是根据定义的字符串长度分配足够的空间。存储CHAR值时，会删除所有末尾空格。CHAR值会根据需要采用空格进行填充以方便比较。

​		CHAR 适合存储很短的字符串，或者所有值都接近同一个长度，以及经常变更的数据。对与非常短的列，CHAR 比 VARCHAR 在存储空间上也更有效率。

> 给VARCHAR分配空间的最好策略是只分配真正需要的空间，慷慨是不明智的。比如VARCHAR(5)和VARCHAR(200)存储 'hellow' 空间开销一样但是更短的列有优势。

##### BLOB 和 TEXT 类型

BLOB 和 TEXT 都是为存储很大的数据而设计的字符串数据类型，分别采用二进制和字符方式存储。

##### 使用枚举 ( ENUM ) 代替字符串类型

可以使用枚举列代替常用的字符串类型，枚举列可以把一些不重复的字符串存储成一个预定义的集合。MySQL 在内部会将每个值在列表中的位置保存为整数，并且在表的 .frm 文件中保存 "数字 - 字符串" 映射关系的 "查找表"。

枚举字段是按照内部存储的整数而不是定义的字符串进行排序的，一种绕过这种限制的方式是按照需要的顺序来定义枚举列。另外也可以在查询中使用FIELD()函数显式地指定排序顺序，但这会导致MySQL无法利用索引消除排序。

枚举字符串列表是固定的，添加或删除字符串必须使用ALTER TABLE。因此对于一系列未来可能会改变的字符串，使用枚举不是好主意，除非能接受只在列表末尾添加元素，这样在MySQL 5.1 后可以不用重建整个表来完成修改。

#### 4.1.4 日期和时间类型

MySQL 提供两种相似的日期类型：DATETIME 和 TIMESTAMP。

##### DATETIME

​		这个类型能保存从1001年到9999年，精度为秒，使用8字节的存储空间。默认情况MySQL 以一种可排序的、无歧义的格式显式DATETIME 值，例如 "2008-01-16 22:25:08"。

##### TIMESTAMP

​		TIMESTAMP 使用4个字节的存储空间，只能表示1970年到2038年。TIMESTAMP提供的值与时区有关系，DATETIME则保留文本表示的日期和时间。

​		TIMESTAMP 默认情况下，如果插入时没有指定第一个 TIMESTAMP 列的值，MySQL 则设置这个列的值为当前时间，在更新一条记录时默认也会更新第一个 TIMESTAMP 列的值 ( 除非在 UPDATE 语句中明确指定了值 )，你可以配置任何 TIMESTAMP 列的插入和更新行为。

除了特殊行为之外，通常也应该尽量使用 TIMESTAMP，它比 DATETIME 空间效率更高。有时候人们会将 Unix 时间戳存储为整数值，但这不会带来任何收益。

#### 4.1.6 选择标识符 ( identifier )

为标识列选择数据类型时应该选择跟关联表中的对应列一样的类型，要精确匹配，包括像UNSIGNED这样的属性。

在可以满足值的范围的需求，并且预留未来增长空间的前提下，应该选择最小的数据类型。

##### 整数类型

​		整数通常是标识列最好的选择，因为它们很快并且可以使用 AUTO_INCREMENT。

##### ENUM 和 SET 类型

​		对于标识列来说，ENUM 和 SET 类型通常是一个糟糕的选择。

##### 字符串类型

​		应避免使用字符串类型作为标识列，因为很消耗空间，并且通常比数字类型慢。对于完全"随机"的字符串，新值会任意分布在很大的空间内，会导致 INSERT 以及一些 SELECT 语句变得很慢。

#### 4.2 MySQL schema 设计中的陷阱

虽然有一些普遍的好或坏的设计原则，但也有一些问题是由MySQL的实现机制导致的，这意味着有可能犯一些只在MySQL下发生的特定错误。

##### 太多的列

​		MySQL 的存储引擎API工作时需要在服务器层和存储引擎层之间通过行缓冲格式拷贝数据，然后在服务器层将缓冲内容解码成各个列，从行缓冲中将编码过的列转换成行数据结构的操作代价是非常高的，转换的代价依赖于列的数量。

##### 太多的关联

​		一个粗略的经验法则，如果希望查询执行得快速且并发性好，单个查询最好在12个表以内做关联。

##### 全能的枚举

​		注意防止过度使用枚举 ( ENUM )。

##### 变相的枚举

​		枚举 ( ENUM ) 列允许在列中存储一组定义值中的单个值，集合 ( SET ) 列则允许在列中存储一组定义值中的一个或多个值，避免混用。

##### 非此发明 （Not Invent Here）的NULL

​		之前写了避免使用 NULL 的好处，并且建议尽可能地考虑替代方案。即使需要存储一个事实上的"空值"到表中时，也不一定非得使用 NULL。也许可以使用0、某个特殊值，或者空字符串作为代替。

​		但是遵循这个原则也不要走极端，当确实需要表示未知值时也不要害怕使用 NULL。

#### 4.3 范式和反范式

在范式化的数据库中，每个事实数据会出现并且只出现一次。相反，在反范式化的数据库中，信息是冗余的，可能会存储在多个地方。

> 用一个很长的字符串作为主键是很糟糕的注意。

#### 4.3.1 范式的优点和缺点

范式化的设计通常能够带来的好处：

- 范式化的更新操作通常要比反范式化要快。
- 当数据较好地范式化时，就只有很少或者没有重复数据，所以只需要修改更少的数据。
- 范式化的表通常更小，可以更好地放在内存里，所以执行操作会更快。
- 很少有多余的数据意味着检索列表数据时更少需要DISTINCT或者GROUP BY语句。

范式化设计的 schema 的缺点是通常需要关联，这不但代价昂贵，也可能使一些索引策略无效。

#### 4.3.2 反范式的优点和缺点

反范式化的 schema 因为所有数据都在一张表中，可以很好地避免关联。

单独的表也能使用更有效的索引策略。

#### 4.3.3 混用范式化和反范式化

完全的范式化和完全的反范式化 schema 都是实验室里才有的东西：在真实世界中很少会这么极端地使用。

最常见的反范式化数据的方法是复制或者缓存，在不同的表中存储相同的特定列。

另一个从父表冗余一些数据到子表的理由是排序的需要。

缓存衍生值也是有用的。

#### 4.4 缓存表和汇总表

有时提升性能最好的方法是在同一张表中保存衍生的冗余数据，有时也需要创建一张完全独立的汇总表或缓存表 ( 特别是为满足检索的需求时 )。

我们用术语"缓存表"来表示存储那些可以比较简单地从 schema 其他表获取 ( 但是每次获取的速度比较慢 ) 数据的表 ( 例如， 逻辑上冗余的数据 )。术语"汇总表"表示保存的是使用 GROUP BY 语句聚合数据的表 ( 例如，数据不是逻辑上冗余的 )。

在使用缓存表和汇总表时，必须决定是实时维护数据还是定期重建。

当重建汇总表和缓存表时，通常需要保证数据在操作时依然可用，这就需要通过使用"影子表"来实现，"影子表"指的是一张在真实表"背后"创建的表。

#### 4.4.1 物化视图

物化视图实际上是预先计算并且存储在磁盘上的表，可以通过各种各样的策略刷新和更新。MySQL 并不原生支持物化视图，可以使用 Justin Swanhart 的开源工具 Flexviews。

#### 4.4.2 计数器表

如果应用在表中保存计数器，则在更新计数器时可能碰到并发问题，使得业务只能串行执行。要获得更高的并发更新性能，可以将计数器保存在多行中，每次随机选择一行进行更新，要获得统计结果SUM()所有行。

> 为了提升读查询的速度，经常会需要建一些额外索引，增加冗余列，甚至是创建缓存表和汇总表。这些方法会增加写查询的负担，也需要额外的维护任务，但在设计高性能数据库时，这些都是常见的技巧：虽然写操作变得更慢了，但是显著地提高了读操作的性能。
>
> 然而，写操作变慢并不是读操作变得更快所付出的唯一代价，还可能同时增加了读操作和写操作的开发难度。

#### 4.5 加快 ALTER TABLE 操作的速度

MySQL 的 ALTER TABLE 操作的性能对大表来说是个大问题。MySQL 执行大部分修改表结构操作的方法是用新的结构创建一个空表，从旧表中查出所有数据插入新表，然后删除旧表。

两种使用技巧：一种是先在一台不提供服务的机器上执行ALTER TABLE操作，然后和提供服务的主库进行切换；另外一种是"影子拷贝"，即用要求的表结构创建一张和源表无关的新表，然后通过重命名和删表操作交换两张表。

不是所有的ALTER TABLE操作都会引起表重建。例如改变或者删除一个列的默认值有两种方法：

```mysql
mysql> ALTER TABLE sakila.film
		-> MODIFY COLUMN rental_duration TINYINT(3) NOT NULL DEFAULT 5; 
-- 这个语句会拷贝整张表到一张新表
mysql> ALTER TABLE sakila.film
		-> ALTER COLUMN rental_duration SET DEFAULT 5;
-- 理论上MySQL可以跳过创建新表的步骤，列的默认值实际上存在表的.frm文件中，可以直接修改这个文件而不需要改动表本身，上面这个语句会直接修改.frm文件而不涉及表数据，所以操作是非常快的。
```

#### 4.5.1 只修改 .frm 文件

ps: 以下技巧是不受官方支持的，慎用。

下面这些操作是有可能不需要重建表的：

- 移除 ( 不是增加 ) 一个列的 AUTO_INCREMENT 属性。
- 增加、移除，或更改 ENUM 和 SET 常量。如果移除的是已经有行数据用到其他值的常量，查询会返回一个空字符串。

基本的技术是为想要的表结构创建一个新的 .frm 文件，然后用它替换掉已经存在的那张表的 .frm 文件。

#### 4.5.2 快速创建 MyISAM 索引

为了高效地载入数据到 MyISAM 表中，有一个常用的技巧是先禁用索引、载入数据，然后重新启用索引：

```mysql
mysql> ALTER TABLE test.load_data DISABLE KEYS;
-- load the data
mysql> ALTER TABLE test.load_data ENABLE KEYS;
```

这个技巧能发挥作用是因为构建索引的工作被延迟到数据完全载入后，这个时候已经可以通过排序来构建索引了，这样会快很多，并且使得索引树的碎片更少、更紧凑。但是对唯一索引无效。

#### 4.6 总结

良好的 schema 设计原则是普遍适用的，但 MySQL 有它自己的实现细节要注意。概括来说，尽可能保持任何东西小而简单总是好的。MySQL 喜欢简单，需要使用数据库的人应该也同样会喜欢简单的原则：

- 尽量避免过度设计，例如会导致及其复杂查询的 schema 设计，或者有很多列的表设计 (很多的意思是介于有点多和非常多之间)。
- 使用小而简单的合适数据类型，除非真实数据模型中有确切的需要，否则 应该尽可能地避免使用 NULL 值。
- 尽量使用相同的数据类型存储相似或相关的值，尤其是要在关联条件中使用的列。
- 注意可变长字符串，其在临时表和排序时可能导致悲观的按最大长度分配内存。
- 尽量使用整型定义标识列。
- 避免使用 MySQL 已经遗弃的特性，例如指定浮点数的精度，或者整数的显示宽度。
- 小心使用 ENUM 和 SET。虽然它们用起来很方便，但是不要滥用，否则有时候会变成陷阱。最好避免使用 BIT。

范式是好的，但是反范式 (大多数情况下意味着重复数据) 有时也是必需的，并且能带来好处。预先计算、缓存或生成汇总表也可能获得很大的好处。

最后，ALTER TABLE 是让人痛苦的操作，因为在大部分情况下，它都会锁表并且重建整张表。我们展示了一些特殊的场景可以使用骇客方法；但是对大部分场景，必须使用其他更常规的方法，例如在备机执行ALTER并在完成后把它切换为主库。