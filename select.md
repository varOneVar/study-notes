## select

**select [all|distinct] select_expr from -> where -> group by [合计函数] -> having -> order by -> limit**

* select_expr

  	1.  *
   	2.  express （计算公式，函数调用，字段  ）
   	3.  as 为字段起别名，多个逗号分隔

* from 子句

  1. as 为表起别名，多个逗号分隔
  2. 多表名，逗号分隔

* where 子句

  1. 从from获得的数据源中筛选, int 1表示真，0表示假

* group by 子句， 分组子句

  1. 分组后会排序：升序ASC(默认)，降序DESC,
  2. 合计函数需要配合group by使用
     1. count 返回不同的非NULL值数目, count(*), count(字段)
     2. sum 求和
     3. max min 最大最小值
     4. avg 平均值
     5. group_concat 返回带有来自一个组的连接的非NULL值字符串结果。组内字符串连接

* having 子句，条件子句

  1. 与where功能用法相同，执行时机不同
     1. where在开始执行监测，对原数据过滤
     2. having对筛选的结果再次过滤
     3. having字段必须是查询出来的，where字段必须是数据表存在的
     4. where不可以使用字段的别名，having可以
     5. where不可以使用合计函数，having可以
     6. SQL标准要求having必须引用group by子句在的列或合计函数中的列

* order by 子句，排序子句

  1. order by 排序字段/别名 排序方式 [, 排序字段/别名 排序方式]...

* limit 子句，限制结果数量子句

  1. 仅对处理好的结果进行数量限制。结果看作一个集合，按出现顺序从0开始
  2. limit 起始位置, 获取条数。省略第一个参数表示从索引0开始。limit 获取条数

* distinct，all 选项

  1. distinct 去重重复记录
  2. 默认为all，全部记录

*  (SELECT ... ) UNION [ALL|DISTINCT] (SELECT ...)

  1. 将多个select结果组合成一个结果集合，默认distinct，order by排序时，需加上limit结合，各个select查询字段列表数量类型应一致，结果字段名以第一条select为准

* 子查询

  1. 子查询需要括号包裹
  2. from型
     1. from后要求是一个表，必须给子查询结果取别名
     2. select * from (select * from tb where id>0) as subfrom where id >1;
  3. where 型
     1. 无需别名，子查询返回一个值，标量子查询
     2. select * from tb where money = （select max(money) from tb);
     3. 列子查询
        1. 如果子查询返回的是一列，使用in或not in完成查询exists 和 not exists 条件，如果子查询返回数据，则返回1或0，常用于判断条件
        2. select column1 from t1 where exists (select * form t2);
     4. 行子查询
        1. 查询条件是一个行
        2. select * from t1 where (id, gender) in (select id, gender from t2);
        3. 行构造符：(col1, col2, ...) 或 ROW(col1, col2, ...)
        4. 行构造符通常用于与对能返回两个或以上列的子查询进行比较。
     5. 特殊运算符
        1.  != all()  相当于 not in
        2.  = some() 相当于 in. any是some的别称
        3.  != some()  不等同于 not in。 不等于其中某一个
        4. all,some可以配合其他运算符一起使用

* join 连接查询

  * [图解join](https://wenku.baidu.com/view/90c126c76f1aff00bed51eea.html)
* [详解join](http://www.zsythink.net/archives/1105/)
  
1. 将多个表字段进行连接，可以指定连接条件
  
  2. 外连接（outer join）
  
     1. 如果数据不存在，也会出现在连接结果里
     2. 左外连接 left join
        1. 如果数据不存在，左表记录会出现，而右表为null填充
   3. 右外连接 right join
      1. 如果数据不存在，右表记录会出现，而左表为null填充
  
  3. 自然连接（natural join）
  
   1. 自动判断连接条件完成连接，相当于省略了using,自动查找相同字段名
   2. natural join, natural left join, natural right join
  
  4. 内连接（inner join,默认，可省略inner）
  
     1. 只有数据存在才能发送连接，不能出现空行
     2. on表示连接条件，条件表达式与where类型，也可以使用where，另外可以使用using，但需字段名相同，using(字段名)；
   3. cross join，交叉连接，即没有条件的内链接
      1. select * from tb1 cross join tb2;

   

   

   

   

   

   

​     

​     