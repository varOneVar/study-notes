  1增

1.1【插入单行】
insert [into] <表名> (列名) values (列值)
例：insert into Strdents (姓名,性别,出生日期) values ('开心朋朋','男','1980/6/15')


1.2【将现有表数据添加到一个已有表】
insert into <已有的新表> (列名) select <原表列名> from <原表名>
例：insert into tongxunlu ('姓名','地址','电子邮件')
select name,address,email
from Strdents


1.3【直接拿现有表数据创建一个新表并填充】
select <新建表列名> into <新建表名> from <源表名>
例：select name,address,email into tongxunlu from strdents


1.4【使用union关键字合并数据进行插入多行】
insert <表名> <列名> select <列值> union select <列值>
例：insert Students (姓名,性别,出生日期)
select '开心朋朋','男','1980/6/15' union（union表示下一行）
select '蓝色小明','男','19**/**/**'

\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

2删

2.1【删除<满足条件的>行】
delete from <表名> [where <删除条件>]
例：delete from a where name='开心朋朋'（删除表a中列值为开心朋朋的行）


2.2【删除整个表】
truncate table <表名>
truncate table tongxunlu
注意：删除表的所有行，但表的结构、列、约束、索引等不会被删除；不能用语有外建约束引用的表


\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

3改

update <表名> set <列名=更新值> [where <更新条件>]
例：update tongxunlu set 年龄=18 where 姓名='蓝色小名'  



distinct name,id 这样的mysql 会认为要过滤掉name和id两个字段都重复的记录，如果sql这样写：select id,distinct name from user，这样mysql会报错，因为distinct必须放在要查询字段的开头。

所以一般distinct用来查询不重复记录的条数。



[time](https://juejin.im/post/58a86dff61ff4b0062b8a86a)