##  简单的增删改查命令egg-mysql

### 增

``` js
# API: insert(table_name, data);
const result = await app.mysql.insert('table_name', {title: 'hellow world'});
const insertSuccess = result.affectedRows === 1;
```

### 查

```js
# API: select('table_name',options)
const result = await app.mysql.select('table_name', {
    where: { status: 'draft'},
    orders: [['created_at', 'des'], ['id', 'des']],
    limit: 10,
    offset: 0
})
# API2: get(table_name, affected_field)
const result = await app.mysql.get('table_name', {id: 2})
```

### 改

```js
# API: update('table_name', row)
// 通过主key去更新数据
const row = {
    id: 123,
    name: 'refresh',
    otherField: 'other field value',
    update_time: app.mysql.literals.now, // 服务端的now()
}
const result = await app.mysql.update('table_name',row);
const updateSuccess = result.affectedRows === 1;
    
```

### 删

```js
# API: delete('table_name', affected_field)
const result = app,mysql.delete('table_name', {name: 'delete man'})
```

### SQL splicing

``` js
# API: query('sql_word')
const results = await app.mysql.query('update table_name set hits = (hits + ?) where id = ?', [1, postID]);
```

### 事务

```js
# API: beginTransaction, commit, rollback
const conn = await app.mysql.beginTransaction();

try {
    await conn.insert('table_name', row1);
    await conn.update('table_name', row2);
    await conn.commit();
} catch (err) {
    await conn.rollback();
    throw err;
}
```

### 带范围的事务

```js
# API: beginTransactionScope(scope,ctx)
const result = await app.mysql.beginTransactionScope(async function (conn) {
    await conn.insert('table_name', row1);
    await conn.update('table_name', row2);
    return { success: true};
}, ctx);
// ctx为当前请求上下文，如果在scope内有error，将自动rollback
```

### MySQL语句

![表达式](../image/248b3188d2cc93f2a827179fd79407c5)

[mysql混杂](https://juejin.im/entry/58e6367ab123db15eb85323b)

[mysql命令](https://juejin.im/post/5ae55861f265da0ba062ec71)

[mysql select](https://juejin.im/entry/590e939444d904007be4da37)



```js 
# API: SELECT 列名称 FROM 表名称 WHERE 条件
查询名字带王字的：select * from students where name like %王%;
查询ID小于5且年龄大于20: select * from students where id < 5 and age > 20;

`or`: 或语句 where city = '上海' or name = '小马'
`like`: 模糊查询 where name like %王%
`in`: where city in ('北京', '上海', '广州', '深圳')
`between and`: 包括两端 where id between 2 and 5 等同于 where id >= 2 and id <= 5
`not`: 搭配其他条件 not like, not in, not null;
`distinct`: 去重
`count(*)` 统计， count(distinct positionId)
`order by [desc]`: 加desc是降序， order by age desc
```

