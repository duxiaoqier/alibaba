# 异常处理，防止 NPE
1.	返回类型为包装数据类型，有可能是 null，返回 int 值时注意判空。 
    - ~~反例~~：public int f(){ return Integer} 对象如果为null，自动解箱抛 NPE。 
2.	数据库的查询结果可能为 null。 
3.	集合里的元素即使 isNotEmpty，取出的数据元素也可能为 null。 
4.	远程调用返回对象，一律要求进行 NPE 判断。 
5.	对于 Session 中获取的数据，建议 NPE 检查，避免空指针。
6.	级联调用 obj.getA().getB().getC()；一连串调用，易产生 NPE。
7.	方法的返回值可以为 null，不强制返回空集合，或者空对象等，必须添加注释充分 说明什么情况下会返回 null 值。调用方需要进行 null 判断防止 NPE 问题。 
    - ==说明==：本规约明确防止NPE是调用者的责任。即使被调用方法返回空集合或者空对象，对调用者来说，也并非高枕无忧，必须考虑到远程调用失败，运行时异常等场景返回null 的情况。
# 日志打印
1. 对 trace/debug/info级别的日志输出，必须使用条件输出形式或者使用占位符的方式。 
    - ==说明==：

        ```
        logger.debug("Processing trade with id: " + id + " symbol: " + symbol);
        ```

        如果日志级别是 warn，上述日志不会打印，但是会执行字符串拼接操作，如果 symbol 是对象，会执行 toString()方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。 
    - 正例：（条件） 
    
        ```
        if (logger.isDebugEnabled()) {
                logger.debug("Processing trade with id: " + id + " symbol: " + symbol);
            }
        ```

    - 正例：（占位符） 

        ```
        logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol);
        ```

2. 异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么往上抛。
    - 正例：
        ```
        logger.error(各类参数或者对象 toString + "_" + e.getMessage(), e);
        ```

3. 避免重复打印日志，浪费磁盘空间，务必在 log4j.xml 中设置 additivity=false。
    - 正例： 
        ```
        <logger name="com.goujianwu" level="INFO" additivity="false"/>
        ```

# mysql建表
1.	表达是与否概念的字段，必须使用 is_xxx 的方式命名，数据类型是 unsigned tinyint （ 1 表示是，0 表示否）
    - ==说明==：任何字段如果为非负数，必须是unsigned。
2.	禁用保留字，如 desc、range、match、delayed 等，参考官方保留字。
3.	唯一索引名为 uk_字段名；普通索引名则为 idx_字段名。 
    - ==说明==：uk_ 即 unique key；idx_ 即 index 的简称。
4.	小数类型为 decimal，禁止使用 float 和 double。 
    - ==说明==：float 和double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储。
5.	如果存储的字符串长度几乎相等，使用 CHAR 定长字符串类型。
6.	表必备三字段：id, gmt_create, gmt_modified。 
    - ==说明==：其中 id 必为主键，类型为 unsigned bigint、单表时自增、步长为 1。gmt_create, gmt_modified 的类型均为 date_time 类型。
7.	在 varchar 字段上建立索引时，必须指定索引长度，没必要对全字段建立索引，根据 实际文本区分度决定索引长度。 
    - ==说明==：索引的长度与区分度是一对矛盾体，一般对字符串类型数据，长度为 20 的索引，区分度会高达 90%以上，可以使用 count(distinct left(列名, 索引长度))/count(*)的区分度来确定。
8.	如果有 order by 的场景，请注意利用索引的有序性。order by 最后的字段是组合索引的一部分，并且放在索引组合顺序的最后，避免出现 file_sort 的情况，影响查询性能。 
    - 正例：where a=? and b=? order by c; 索引：a_b_c 
    - ==反例==：索引中有范围查找，那么索引有序性无法利用，如：WHERE a>10 ORDER BY b; 索引 a_b 无法排序。
9.	利用覆盖索引来进行查询操作，来避免回表操作。 
    - ==说明==：如果一本书需要知道第 11 章是什么标题，会翻开第 11 章对应的那一页吗？目录浏览一下就好，这个目录就是起到覆盖索引的作用。 
    - 正例：IDB 能够建立索引的种类：主键索引、唯一索引、普通索引，而覆盖索引是一种查询的一种效果，用 explain 的结果，extra 列会出现：using index.
    [一个例子](http://www.searchdatabase.com.cn/showcontent_53221.htm)
10.	利用延迟关联或者子查询优化超多分页场景。 
    - ==说明==：MySQL 并不是跳过 offset 行，而是取 offset+N 行，然后返回放弃前 offset 行，返回 N 行，那当 offset 特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特 定阈值的页数进行 SQL 改写。 
    - 正例：先快速定位需.获取的 id 段，然后再关联： SELECT a.* FROM 表 1 a, (select id from 表1 where 条件 LIMIT 100000,20) b where a.id = b.id 
11.	建组合索引的时候，区分度最高的在最左边。 
    - 正例：如果 where a=? and b=? ，a 列的几乎接近于唯一值，那么只需要单建 idx_a 索引即可。 
    - ==说明==：存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如：where a>? and b=? 那么即使a的区分度更高，也必须把b放在索引的最前列。

# SQL规约
1.	不要使用 count(列名)或 count(常量)来替代 count(*)，count(*)就是SQL92定义的 标准统计行数的语法，跟数据库无关，跟NULL和非NULL无关。 
    - ==说明==：count(*)会统计值为 NULL 的行，而 count(列名)不会统计此列为 NULL 值的行。
2.	count(distinct col) 计算该列除 NULL 之外的不重复数量。注意count(distinct col1, col2) 如果其中一列全为 NULL，那么即使另一列有不同的值，也返回为 0。
3.	当某一列的值全是 NULL 时，count(col)的返回结果为 0，但sum(col)的返回结果为 NULL，因此使用 sum()时需注意 NPE 问题。 
    - 正例：可以使用如下方式来避免 sum 的 NPE 问题：
    
        ```
        SELECT IF(ISNULL(SUM(g)),0,SUM(g)) FROM table
        ```

4.	使用 ISNULL()来判断是否为 NULL 值。注意：NULL 与任何值的直接比较都为 NULL。 
    - ==说明==： 
        - NULL<>NULL 的返回结果是 NULL，不是 false。
        - NULL=NULL 的返回结果是 NULL，不是 true。 
        - NULL<>1 的返回结果是 NULL，而不是 true。
5.	在代码中写分页查询逻辑时，若count 为0应直接返回，避免执行后面的分页语句。
6.	不得使用外键与级联，一切外键概念必须在应用层解决。 
    - ==说明==：（概念解释）学生表中的 student_id 是主键，那么成绩表中的 student_id 则为外键。 如果更新学生表中的 student_id，同时触发成绩表中的 student_id 更新，则为级联更新。外键与级联更新适用于单机低并发，不适合分布式、高并发集群；级联更新是强阻塞，存在数据库更新风暴的风险；外键影响数据库的插入速度。
7.	in操作能避免则避免，若实在避免不了，需要仔细评估in后边的集合元素数量，控 制在1000个之内。
8.	TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少，但TRUNCATE 无事务且不触发trigger，有可能造成事故，故不建议在开发代码中使用此语句。 
    - 说明：TRUNCATE TABLE 在功能上与不带 WHERE 子句的 DELETE 语句相同。
9.	禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。
