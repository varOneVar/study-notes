> mysql的基础使用

#### 数据类型

~~~mysql
# 括号里的3表示显示的宽度，与占用字节无关，当设置字段属性ZEROFILL时，会在未达到要求的数字前补足0，比如存入2，显示的是002。不会影响实际存储值，假如存入4399，还是会显示4399，不会因为Int(3)而缺少
# 一般数字都是设置的INT类似，占用3字节，无符号(UNSIGNED)能表示四十多亿(2的32次方减1)
INT(3)

# 占用1字节，最大存255，表示性别，单科成绩足够了
TINYINT 

# 8字节，到9999年，有时差。高版本(5.6.4)支持小数秒，括号里能写0-6，表示秒后小数位数，0表示精确到秒，3表示精确到微秒。高版本还进行了优化，没有使用小数秒时，占用8字节变成了占用5字节。有种存日期的方式，存数字201908192316表示2019年8月19号23点16分，占用字节少又没时差。
DATETIME(3) # 5.6版本支持设置默认值 CURRENT_TIMESTAMP 或者 ON UPDTAE CURRENT_TIMESTAMP(前者在插入数据时会自动设置当前时间，设置两个有时会报错)

# 时间戳，无时差，4字节，但是只能表示到2038年1月19号
TIMESTAMP(0)

# 字符串一般存的可变字符,括号里表示字符限制个数，这里限制存入255个字符，编码一个字符占用字节是根据字符集来的，utf8编码汉字3字节，utf8mb4编码汉字4字节(常用)，实际占用字节数就是4乘以字符个数+2，这里加的2是记录占用字节数所需要的字节(或+1)。占用空间根据存储字符个数以及字符集类型而变化
# 一行包含的所有列的总数据大小最大不能超过65535个字节，所以该字段能占用多少需要掂量，文本比较多最好还是使用TEXT类型
VARCHAR(255)

# 定长字符，最多255字符，默认1，char(0)只能存''和NULL，不足长度用空格补足，少用
CHAR(18)

# 不受《一行包含的所有列的总数据大小最大不能超过65535个字节》影响，最大占用65535字节（2的16次方减1），或者使用LONGTEXT(2的32次方减1)
TEXT
~~~



#### 库

~~~mysql
# 展示
SHOW DATABASES
# 创建, 添加IF NOT EXISTS可以防止报错，已存在表再创建就会报错。
CREATE DATABASE [,IF NOT EXISTS] db_name
# 切换数据库
USE db_name
# 删除数据库,添加IF EXISTS防止报错，删除不存在的库会报错。
DROP DATABASE [,IF EXISTS] db_name
~~~

#### 表

~~~mysql
# 展示
SHOW TABLES

# 查看表结构
DESCRIBE 表名;
DESC 表名;
EXPLAIN 表名;
SHOW COLUMNS FROM TABLE 表名;
SHOW FIELDS FROM TABLE 表名;
SHOW CREATE TABLE 表名\G # 查看建表语句, \G可以换分号。
# 表名可以换成 【库名.表名】, 可以在任意表里使用，单独表名只能在本表内查看本表；

# 创建, 方括号为可选
CREATE TABLE [,IF NOT EXISTS] tb_name (
   列名1  数据类型  [列属性，多个空格分块] COMMENT '列注释',
   列名2	数据类型  [列属性，多个空格分块] COMMENT '列注释',
   列名3	数据类型  [列属性，多个空格分块] COMMENT '列注释'
) COMMENT '表注释'

# 删除, 可批量，方括号可选
DROP TABLE [,IF EXISTS] 表1, 表2, 表3, ..., 表n;

# 修改表名
ALTER TABLE 旧表名 RENAME TO 新表名;
RENAME TABLE 旧表名1 TO 新表名1, 旧表名2 TO 新表名2, 旧表名3 TO 新表名3; # 多表操作
# 如果指定了库名，可以产生转移表的效果。

# 增加列 最后的【first】 或者 【after 列名】表示插入表的位置
ALTER TABLE 表名 ADD COLUMN 字段名 数据类型 [列属性] [FIRST或者AFTER 列名];

# 删除列
ALTER TABLE 表名 DROP COLUMN 列名;

# 修改列信息，修改后的数据类型和属性一定要兼容表中现有的数据
ALTER TABLE 表名 MODIFY 列名 新数据类型 [新列属性];
ALTER TABLE 表名 CHANGE 旧列名 新列名 新数据类型 [新属性]; # 修改列性质同时修改列名

# 修改列的位置
ALTER TABLE 表名 MODIFY 列名 列的类型 列的属性 FIRST;
ALTER TABLE 表名 MODIFY 列名 列的类型 列的属性 AFTER 指定列名;

# 多操作空格分块
ALTER TABLE 表名 操作1, 操作2, ..., 操作n;

~~~

#### 列属性

~~~mysql
# 批量插入
INSERT INTO 表名(列1, 列2) VALUES(列值1, 列值2),(列值1, 列值2),(列值1, 列值2),(列值1, 列值2);

# 默认值
列名 列类型 DEFAULT 默认值

# NOT NULL，必填字段
列名 列的类型 NOT NULL

# 主键，每个表只有一个主键，且带有 NOT NULL特性
列名 列的类型 PRIMARY KEY
PRIMARY KEY (列名1, 列名2, ...) # 复合主键,防止相同，将多列字段组合成一个主键，字段长度和字段数目要越少越好。复合主键只能用这种方式写。

# UNIQUE，约束。保证字段值唯一性,主键也算是一个约束，它的名字就是默认的PRIMARY。不写名称系统自动添加名称
列名 列的类型 UNIQUE
列名 列的类型 UNIQUE KEY
UNIQUE [约束名称](列名1, 列名2, ...)
UNIQUE KEY [约束名称](列名1, 列名2, ...)
# 主键唯一，unique可以多个。主键不为NULL，unique支持NULL;

# 外键,关联到外表必须有这个字段才能存入，一种约束。系统会自动添加外键名称
CONSTRAINT [外键名称] FOREIGN KEY(列1, 列2) REFERENCES 父表名(列1, 列2);

# AUTO_INCREMENT 自增长属性。唯一存在，一般做主键，必须有索引，不能设默认值
列名 列的类型 AUTO_INCREMENT
ALTER TABLE 表名 AUTO_INCREMENT = 1; # 重置id操作;

# UNSIGNEN 和 ZEROFILL, 作用于INT
# 多个属性空格分开
# 字段名有空格用反引号包裹``, 最好是别设置这样的字段名。
# 字段名别用保留字create,database,int之类的，如果非要用，也要用反引号包裹；

~~~

#### 查询

~~~mysql
# 少用*, 写出需要的字段；
# 同名字段可以重复写；
# AS 可以省略；

# 去重可以DISTINCT或GROUP BY，distinct只能写一个在字段前，表示两条结果的每一个列中的值都相同，就算重复
SELECT DISTINCT 列名1, 列名2 FROM 表名;

# 限制条数，页数从0开始，其实就是查询数据开始行。只写一个参数表示限制条数
LIMIT 页数,条数

# 排序，升|降，默认asc，多个排序时，从左到右一个个排好。
ORDER BY 列1 ASC|DESC, 列2 ASC|DESC

# <>为不等于

# 多字段in操作
(列1,列2) IN (('值1', '值2'),('值1', '值2'),('值1', '值2'),('值1', '值2'));

# 匹配NULL，不能直接使用普通的操作符来与NULL值进行比较，必须使用IS NULL或者IS NOT NULL
IS NULL # 返回bool
IS NOT NULL # 返回bool
IFNULL(列名,替换值) # 使用concat函数查询时，要对可能是null的字段使用ifnull,不然会报错

# AND操作符的优先级高于OR操作符，也就是说在判断某条记录是否符合条件时会先检测AND操作符两边的搜索条件
score > 95 OR score < 55 AND subject = '论萨达姆的战争准备'
 =>   score > 95    和    score < 55 AND subject = '论萨达姆的战争准备'   两段
 
 
# like 和 not like模糊查询匹配
# %: 代表任意一个字符串
# _: 代表任意一个字符
# 转义通配符： \%, \_

# a DIV b : 除法，取商的整数部分,/会保留商的小数部分

# a XOR b : a和b有且只有一个为真，表达式为真

# 对字段直接使用表达式，下面查出的score都会加上100，注意查出来的字段名也会变score + 100
# 搜索时，使用+会把值转化为数字再计算，字符串会只会选开头为数字的部分，像parseInt函数那样，但是如果开头也是字符串，就会转化为0。比如 '12asdd' => 12, 'asd' => 0;
SELECT  number, subject, score + 100 FROM student_score; 

# count(*) 与 count(列) 区别在于前者会统计值为NULL的行;

# 不在搜索条件中的那些记录是不参与统计的

# 统计函数去重
SELECT COUNT(DISTINCT major) FROM student_info;

# 存储数据时，mysql隐式把某个值转换为某个列需要的类型

# 分组查询: 针对某个列，将该列的值相同的记录分到一个组中。
# 分组查询执行过程: 对group by 后的字段分组，然后 分别对每个分组中记录的使用聚合函数的列调用聚合函数进行数据统计
# group by 后跟多个字段是嵌套分组，从左至右不断对上一个分组进行分组，聚合函数作用于最后一个字段。
# 分组查询必须有聚合函数。分组查询只能有分组字段和聚合字段在查询列表。
# 搜索条件执行过滤后再进行分组，聚合函数作用于分组，所有where子句不能使用聚合函数，除非where子句过滤条件里是子查询语句。
# 对分组结果使用过滤用having，having可以调用聚合函数过滤。
# 分组列中有NULL时，也会对NULL进行分组
# 语句顺序
SELECT [DISTINCT] 查询列表
[FROM 表名]
[WHERE 布尔表达式]
[GROUP BY 分组列表 ]
[HAVING 分组过滤条件]
[ORDER BY 排序列表]
[LIMIT 开始行, 限制条数]

#sad
~~~

