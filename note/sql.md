###  SQL优化技巧

**一、一些常见的****SQL****实践**

**（****1****）负向条件查询不能使用索引**

- ```mssql
  `select * from order where status!=0 and stauts!=1`
  ```

  

not in/not exists都不是好习惯

可以优化为in查询：

- select * from order where status in(2,3)

 

**（****2****）前导模糊查询不能使用索引**

- select * from order where desc like '%XX'

而非前导模糊查询则可以：

- select * from order where desc like 'XX%'

 

**（****3****）数据区分度不大的字段不宜使用索引**

- select * from user where sex=1

原因：性别只有男，女，每次过滤掉的数据很少，不宜使用索引。

经验上，能过滤80%数据时就可以使用索引。对于订单状态，如果状态值很少，不宜使用索引，如果状态值很多，能够过滤大量数据，则应该建立索引。

 

**（****4****）在属性上进行计算不能命中索引**

- select * from order where YEAR(date) < = '2017'

即使date上建立了索引，也会全表扫描，可优化为值计算：

- select * from order where date < = CURDATE()

或者：

- select * from order where date < = '2017-01-01'

 

**二、并非周知的****SQL****实践**

**（****5****）如果业务大部分是单条查询，使用****Hash****索引性能更好，例如用户中心**

- select * from user where uid=?
- select * from user where login_name=?

原因：

B-Tree索引的时间复杂度是O(log(n))

Hash索引的时间复杂度是O(1)

 

**（****6****）允许为****null****的列，查询有潜在大坑**

单列索引不存null值，复合索引不存全为null的值，如果列允许为null，可能会得到“不符合预期”的结果集

- select * from user where name != 'shenjian'

如果name允许为null，索引不存储null值，结果集中不会包含这些记录。

所以，请使用not null约束以及默认值。

 

**（****7****）复合索引最左前缀，并不是值****SQL****语句的****where****顺序要和复合索引一致**

用户中心建立了(login_name, passwd)的复合索引

- select * from user where login_name=? and passwd=?
- select * from user where passwd=? and login_name=?

**都能够命中索引**

 

- select * from user where login_name=?

**也能命中索引**，满足复合索引最左前缀

 

- select * from user where passwd=?

**不能命中索引**，不满足复合索引最左前缀

 

**（****8****）使用****ENUM****而不是字符串**

ENUM保存的是TINYINT，别在枚举中搞一些“中国”“北京”“技术部”这样的字符串，字符串空间又大，效率又低。

 

**三、小众但有用的****SQL****实践**

**（****9****）如果明确知道只有一条结果返回，****limit 1****能够提高效率**

- select * from user where login_name=?

可以优化为：

- select * from user where login_name=? limit 1

原因：

你知道只有一条结果，但数据库并不知道，明确告诉它，让它主动停止游标移动

 

**（****10****）把计算放到业务层而不是数据库层，除了节省数据的****CPU****，还有意想不到的查询缓存优化效果**

- select * from order where date < = CURDATE()

这不是一个好的SQL实践，应该优化为：

$curDate = date('Y-m-d');

$res = mysql_query(

  'select * from order where date < = $curDate');

原因：

释放了数据库的CPU

多次调用，传入的SQL相同，才可以利用查询缓存

 

**（****11****）强制类型转换会全表扫描**

- select * from user where phone=13800001234

**你以为会命中****phone****索引么？大错特错**了，这个语句究竟要怎么改？

 

末了，再加一条，不要使用select *（潜台词，文章的SQL都不合格 =_=），只返回需要的列，能够大大的节省数据传输量，与数据库的内存使用量哟。

 

(12)mysql分页优化

一般刚开始学SQL的时候，会这样写  

SELECT * FROM table ORDER BY id LIMIT 1000, 10;  

但在数据达到百万级的时候，这样写会慢死  

SELECT * FROM table ORDER BY id LIMIT 1000000, 10;  

也许耗费几十秒 

网上很多优化的方法是这样的  

SELECT * FROM table WHERE id >= (SELECT id FROM table LIMIT 1000000, 1) LIMIT 10;  

是的，速度提升到0.x秒了，看样子还行了 

可是，还不是完美的！  

以下这句才是完美的！  

SELECT * FROM table WHERE id BETWEEN 1000000 AND 1000010;  

比上面那句，还要再快5至10倍  

另外，如果需要查询 id 不是连续的一段，最佳的方法就是先找出 id ，然后用 in 查询  

SELECT * FROM table WHERE id IN(10000, 100000, 1000000...);  

(13)in多个字段的时候如果2个字段占据数据超过数据总量的大概百分30 就不会走索引