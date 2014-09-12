## Model

### 原型方法

#### field(field, reverse)

设置要查询的字段。

* `field` string | array 要查询的字段，可以是字符串，也可以是数组
* `reverse` boolean 是否反选字段
* `return` this

```js
// 生成的sql语句为，以下同： 
// SELECT * FROM `meinv_group` 
D('Group').field().select(); 
// SELECT `id`,`title` FROM `meinv_group` 
D('Group').field('id, title').select(); 
// SELECT `id`,`title` FROM `meinv_group` 
D('Group').field(['id', 'title']).select();
// SELECT `cate_id`,`md5`,`width`,`height`,`pic_nums`,`view_nums`,`date` FROM `meinv_group` 
D('Group').field(['id', 'title'], true).select();
```


#### limit(offset, length)

设置查询的数量。

* `offset` 起始位置
* `length` 查询的数目
* `return` this

```js
// SELECT * FROM `meinv_group` LIMIT 10 
D('Group').limit(10).select();
// SELECT * FROM `meinv_group` LIMIT 10,20 
D('Group').limit(10, 20).select();
```

#### page(page, listRows)

设置当前查询的页数，页数从1开始

* `page` 当前的页数
* `listRows` 一页多少条记录，默认值为`C('db_nums_per_page')`
* `return` this

```js
// SELECT * FROM `meinv_group` 
D('Group').page().select();
// SELECT * FROM `meinv_group` LIMIT 0,20 
D('Group').page(1).select();
// SELECT * FROM `meinv_group` LIMIT 10,10 
D('Group').page(2, 10).select();
```

#### union(union, all)

联合查询

* `union` 联合查询的字符串
* `all` 是否为UNION ALL模式
* `return` this

```js
// SELECT * FROM `meinv_pic1` UNION (SELECT * FROM meinv_pic2) 
D('Pic1').union('SELECT * FROM meinv_pic2').select();
// SELECT * FROM `meinv_pic1` UNION ALL (SELECT * FROM meinv_pic2)
D('Pic1').union('SELECT * FROM meinv_pic2', true).select();
// SELECT * FROM `meinv_pic1` UNION ALL (SELECT * FROM `meinv_pic2`)
D('Pic1').union({table: 'meinv_pic2'}, true).select();
// SELECT * FROM `meinv_pic1` UNION ALL (SELECT * FROM `meinv_pic2`) UNION (SELECT * FROM meinv_pic3)
D('Pic1').union({table: 'meinv_pic2'}, true).union({table: 'meinv_pic3'}).select();
```


#### join(join)

组合查询

* `join` 可以是字符串、数组、对象
* `return` this


```js
// SELECT * FROM `meinv_group` LEFT JOIN meinv_cate ON meinv_group.cate_id=meinv_cate.id
D('Group').join('meinv_cate ON meinv_group.cate_id=meinv_cate.id').select();

// SELECT * FROM `meinv_group` LEFT JOIN meinv_cate ON meinv_group.cate_id=meinv_cate.id RIGHT JION meinv_tag ON meinv_group.tag_id=meinv_tag.id
D('Group').join(['meinv_cate ON meinv_group.cate_id=meinv_cate.id', 'RIGHT JOIN meinv_tag ON meinv_group.tag_id=meinv_tag.id']).select();

// SELECT * FROM meinv_group INNER JOIN `meinv_cate` AS c ON meinv_group.`cate_id`=c.`id`
D('Group').join({
  table: 'cate', 
  join: 'inner', //join方式，有 left, right, inner 3种方式
  as: 'c', //表别名
  on: ['cate_id', 'id'] //ON条件
}).select();

// SELECT * FROM meinv_group AS a LEFT JOIN `meinv_cate` AS c ON a.`cate_id`=c.`id` LEFT JOIN `meinv_group_tag` AS d ON a.`id`=d.`group_id`
D('Group').alias('a').join({
  table: 'cate',
  join: 'left',
  as: 'c',
  on: ['cate_id', 'id']
}).join({
  table: 'group_tag',
  join: 'left',
  as: 'd',
  on: ['id', 'group_id']
}).select()

// SELECT * FROM meinv_group AS a LEFT JOIN `meinv_cate` AS c ON a.`id`=c.`id` LEFT JOIN `meinv_group_tag` AS d ON a.`id`=d.`group_id`
D('Group').alias('a').join({
  cate: {
    join: 'left', //有left,right,inner 3个值
    as: 'c',
    on: ['id', 'id']
  },
  group_tag: {
    join: 'left',
    as: 'd',
    on: ['id', 'group_id']
  }
}).select()

//SELECT * FROM `meinv_group` LEFT JOIN `meinv_cate` ON meinv_group.`id`=meinv_cate.`id` LEFT JOIN `meinv_group_tag` ON meinv_group.`id`=meinv_group_tag.`group_id`
D('Group')join({
  cate: {
    on: ['id', 'id']
  },
  group_tag: {
    on: ['id', 'group_id']
  }
}).select()

//SELECT * FROM `meinv_group` LEFT JOIN `meinv_cate` ON meinv_group.`id`=meinv_cate.`id` LEFT JOIN `meinv_group_tag` ON meinv_group.`id`=meinv_group_tag.`group_id` LEFT JOIN `meinv_tag` ON (meinv_group.`id`=meinv_tag.`id` AND meinv_group.`title`=meinv_tag.`name`)
D('Group').join({
  cate: {
    on: 'id, id'
  },
  group_tag: {
    on: ['id', 'group_id']
  },
  tag: {
    on: { // 多个字段的ON
      id: 'id',
      title: 'name'
    }
  }
}).select()
```


#### order(order)

设置排序方式

* `order` 排序方式，字符串
* `return` this

```js
// SELECT * FROM `meinv_group` ORDER BY id
D('Group').order('id').select();
// SELECT * FROM `meinv_group` ORDER BY id DESC
D('Group').order('id DESC').select();
// SELECT * FROM `meinv_group` ORDER BY id DESC,title ASC
D('Group').order('id DESC,title ASC').select();
// SELECT * FROM `meinv_group` ORDER BY id ASC
D('Group').order(['id ASC']).select();
// SELECT * FROM `meinv_group` ORDER BY id ASC,title DESC
D('Group').order(['id ASC', 'title DESC']).select()
// SELECT * FROM `meinv_group` ORDER BY `id` ASC,`title` DESC
D('Group').order({id: 'ASC', title: 'DESC'}).select()
```

#### alias(alias)

设置表别名

* `alaias` 表别名，字符串
* `return` this

```js
//SELECT * FROM meinv_group AS a
D('Group').alias('a').select();
```

#### having(str)

having查询

* `str` having查询的字符串
* `return` this

```js
// SELECT * FROM `meinv_group` HAVING view_nums > 1000 AND view_nums < 2000
D('Group').having('view_nums > 1000 AND view_nums < 2000').select();
```

#### group(field)

分组查询

* `field` 设定分组查询的字段
* `return` this

```js
// SELECT * FROM `meinv_group` GROUP BY `view_nums`
D('Group').group('view_nums').select();
```

#### distinct(field)

去重查询

* `field` 去重的字段
* `return` this

```js
// SELECT Distinct  `view_nums` FROM `meinv_group`
D('Group').distinct('view_nums').select();
```

#### where(where)

设置where条件

* `where` 查询条件，可以是字符串、对象
* `return` this

**普通条件**

```js
// SELECT * FROM `meinv_group`
D('Group').where().select();
// SELECT * FROM `meinv_group` WHERE ( `id` = 10 )
D('Group').where({id: 10}).select();

//查询字符串
// SELECT * FROM `meinv_group` WHERE id = 10 OR id < 2
D('GROUP').where('id = 10 OR id < 2').select();

//操作符
// SELECT * FROM `meinv_group` WHERE ( `id` != 10 )
D('Group').where({id: ['!=', 10]}).select(); //这里的操作符有 > >= < <= != 

```

**EXP条件**

thinkjs默认会对字段和值进行转义，防止安全漏洞。有时候一些特殊的情况不希望被转义，可以使用EXP的方式

```js
//SELECT * FROM `meinv_group` WHERE name='name'
D('GROUP').where({name: ['EXP', "='name'"]});
```

```js
//将view_nums字段值加1
D('GROUP').update({view_nums: ['EXP', 'view_nums+1']});
```
**LIKE条件**

```js
// LIKE和NOT LIKE
// SELECT * FROM `meinv_group` WHERE ( `title` NOT LIKE 'welefen' )
D('Group').where({title: ['NOTLIKE', 'welefen']}).select();
// SELECT * FROM `meinv_group` WHERE ( `title` LIKE '%welefen%' )
D('Group').where({title: ['like', '%welefen%']}).select();

//LIKE多个值
// SELECT * FROM `meinv_group` WHERE ( (`title` LIKE 'welefen' OR `title` LIKE 'suredy') )
D('Group').where({title: ['like', ['welefen', 'suredy']]}).select()

//多个字段LIKE同一个值，或的关系
//SELECT * FROM `meinv_group` WHERE ( (`title` LIKE '%welefen%') OR (`content` LIKE '%welefen%') )
D('Group').where({'title|content': ['like', '%welefen%']}).select();
//多个字段LIKE同一个只，与的关系
//SELECT * FROM `meinv_group` WHERE ( (`title` LIKE '%welefen%') AND (`content` LIKE '%welefen%') )
D('Group').where({'title&content': ['like', '%welefen%']}).select();
```


**IN条件**

```js
//SELECT * FROM `meinv_group` WHERE ( `id` IN ('10','20') )
D('Group').where({id: ['IN', '10,20']}).select();
//SELECT * FROM `meinv_group` WHERE ( `id` IN (10,20) )
D('Group').where({id: ['IN', [10, 20]]}).select();
//SELECT * FROM `meinv_group` WHERE ( `id` NOT IN (10,20) )
D('Group').where({id: ['NOTIN', [10, 20]]}).select()
```

**多字段查询**

```js
//SELECT * FROM `meinv_group` WHERE ( `id` = 10 ) AND ( `title` = 'www' )
D('Group').where({id: 10, title: "www"}).select();

//修改为或的关系
//SELECT * FROM `meinv_group` WHERE ( `id` = 10 ) OR ( `title` = 'www' )
D('Group').where({id: 10, title: "www", _logic: 'OR'}).select();

//修改为异或的关系
//SELECT * FROM `meinv_group` WHERE ( `id` = 10 ) XOR ( `title` = 'www' )
D('Group').where({id: 10, title: "www", _logic: 'XOR'})
```

**BETWEEN查询**

```js
//SELECT * FROM `meinv_group` WHERE (  (`id` BETWEEN 1 AND 2) )
D('Group').where({id: ['BETWEEN', 1, 2]}).select();
//SELECT * FROM `meinv_group` WHERE (  (`id` BETWEEN '1' AND '2') )
D('Group').where({id: ['between', '1,2']}).select()
```

**复合查询**
```js
// SELECT * FROM `meinv_group` WHERE ( `id` > 10 AND `id` < 20 )
D('Group').where({id: {
  '>': 10,
  '<': 20
}}).select()

// SELECT * FROM `meinv_group` WHERE ( `id` < 10 OR `id` > 20 )
D('Group').where({id: {
  '<': 10,
  '>': 20,
  _logic: 'OR'
}}).select()

//SELECT * FROM `meinv_group` WHERE ( `id` >= 10 AND `id` <= 20 ) OR ( `title` LIKE '%welefen%' ) OR ( `date` > '2014-08-12' )
D('Group')..where({id: {
  '>=': 10,
  '<=': 20
}, 'title': ['like', '%welefen%'], date: ['>', '2014-08-12'], _logic: 'OR'}).select()
```


#### count(field)

查询符合条件的数目，可以有where条件

* `field` count的字段，默认会从数据表里查找主键的字段
* `return` promise

```js
//SELECT COUNT(id) AS thinkjs_count FROM `meinv_group` LIMIT 1
D('Group').count('id').then(function(count){
    //count为符合条件的数目
});
```

#### sum(field)

对符合条件的字段值求和，可以有where条件

* `field` 求和的字段
* `return` promise

```js
//SELECT SUM(view_nums) AS thinkjs_sum FROM `meinv_group` LIMIT 1
D('Group').sum('view_nums').then(function(sum){
    //sum为求和的值
})
```

#### min(field)

求字段的最小值

* `field` 要求最小值的字段
* `return` promise

```js
//SELECT MIN(view_nums) AS thinkjs_min FROM `meinv_group` LIMIT 1
D('Group').min('view_nums').then(function(min){
    //min为最小的view_nums值
})
```


#### max(field)

求字段的最大值

* `field` 要求最大值的字段
* `return` promise

```js
//SELECT MAX(view_nums) AS thinkjs_max FROM `meinv_group` LIMIT 1
D('Group').max('view_nums').then(function(max){
    //max为最小的view_nums值
})
```

#### avg(field)

求字段的平均值

* `field` 要求平均值的字段
* `return` promise

```js
//SELECT AVG(view_nums) AS thinkjs_avg FROM `meinv_group` LIMIT 1
D('Group').avg('view_nums').then(function(avg){

})
```

#### add(data)

插入数据

* `data` 要插入的数据，对象
* `return` promise

```js
var data = {
    title: 'xxx',
    content: 'yyy'
};
D('Group').add(data).then(function(insertId){
    //如果插入成功，insertId为插入的id
}).catch(function(err){
    //插入失败，err为具体的错误信息
})
```

如果数据表中有字段设置为`unique`，插入一个已经存在的值时就会报错。这种情况一般需要先按这个字段去查询下看有没有对应的记录，如果没有在进行插入。

为了简化开发者的使用，thinkjs提供了thenAdd方法。

#### thenAdd(data, where, returnDetail)

当数据表中不存在where条件对应的数据时，才进行插入。

* `data` 要插入的数据
* `where` 检测的条件
* `returnDetail` 是否返回详细的信息

```js
//假设数据表中字段title类型为UNIQUE
var data = {
    title: 'xxx',
    content: 'yyy'
};
var where = {
    title: 'xxx'
}
D('Group').thenAdd(data, where).then(function(id){
    //id为已经存在的id或者刚插入的id
})
//返回详细的信息
D('Group').thenAdd(data, where, true).then(function(data){
    //data数据结构为
    data = {
        type: 'exist' || 'add', //exist表示之前已经存在，add表示新添加
        id: 111
    }
})
```

使用场景：用户注册时就可以通过该方法来检查用户名或者邮箱已经存在。


#### addAll(data)

一次添加多条数据

* `data` 要添加的数据，数组
* `return` promise

```js
var data = [{title: 'xxx'}, {title: 'yyy'}];
D('Group').addAll(data).then(function(insertId){
    //插入成功
}).catch(function(err){
    //插入失败
})
```

#### delete()

* `return` promise

删除符合条件的数据

```js
//删除所有数据
D('Group').delete().then(function(affectedRows){
    //affectedRows为影响的行数
})
//删除id小于100的数据
D('Group').where({id: ['<', 100]}).delete().then(function(affectedRows){
    //affectedRows为影响的行数
})
```


#### update(data)

更新符合条件的数据

* `data` 要更新的数据
* `return` promise

```js
//将id<10的title设置为空
D('Group').where({id: ['<', 10]}).update({
    title: ''
})
```

#### select()

查询符合条件的数据

* `return` promise

```js
D('Group').where({id: ['IN', [1, 2, 3]]}).select().then(function(data){
  //data为数组
  //如果查询数据为空，那么data为 []
  data = [{
    id: 1, 
    title: '',
    ...
  },{
    id: 2,
    title: ''
    ...
  }
  ]
})
```


#### find()

查找某一条符合条件的数据。

* `return` promise

```js
//查询id=1000的一条数据
D('Group').where({id: 1000}).find().then(function(data){
  //data为一个数据对象
  //如果数据为空，那么data为 {}
  data = {
    id: 1000,
    title: 'xxx',
    ...
  }
})
```

#### updateInc(field, step)

将字段值增加

* `field` 要增加的字段
* `step` 增加的数值，默认为1
* `return` promise

```js
//将id=10的浏览数加1
D('Group').where({id: 10}).updateInc('view_nums').then(function(){

})
//将id=100的浏览数加10
D('Group').where({id: 100}).updateInc('view_nums', 10).then(function(){

})
```

#### updateDec(field, step)

将字段值减少

* `field` 要减少的字段
* `step` 减少的值，默认为1
* `return` promise

```js
//将id=10的浏览数减1
D('Group').where({id: 10}).updateDec('view_nums').then(function(){

})
//将id=100的浏览数减10
D('Group').where({id: 100}).updateDec('view_nums', 10).then(function(){

})
```

#### getField(field, onlyOne)

获取某个字段的值。

* `field` 要获取的字段，可以是一个字段，也可以是多个字段，多个字段用,隔开
* `onlyOne` 是否只需要一个值, true 或者数字
* `return` promise

```js
//取id>5000的集合，只需要id值，不需要其他字段值
D('Group').where({id: ['>',5000]}).getField('id').then(function(data){
  //data为一个数组
  data = [7565,7564,7563,7562,7561,7560,7559,7558,7557]
})
//只需要id>5000的一个值
D('Group').where({id: ['>', 5000]}).getField('id', true).then(function(data){
  //data为数字
  data = 7557;
})
//只需要id>5000的3值
D('Group').where({id: ['>', 5000]}).getField('id', 3).then(function(data){
  //data为数字
  data = [7559, 7558, 7557];
})

//获取id和view_nums 2个字段的值
D('Group').getField('id,view_nums').then(function(data){
  data = {
    "id":[7565,7564,7563,7562,7561,7560,7559,7558,7557],
    "view_nums":[1965,1558,2335,2013,1425,1433,1994,2035,1118]
  }
})
```
 
#### countSelect(options, flag)

* `options` 查询参数
* `flag` 当分页值不合法的时候，处理情况。true为修正到第一页，false为修正到最后一页，默认不进行修正。
* `return` promise

```js
//按每页20条数据来展现文件
D('Article').page(this.get("page"), 20).countSelect().then(function(data){
    //data的数据格式为
    {
        count: 123, //总条数
        total: 10, //总页数
        page: 1, //当前页
        num: 20, //每页显示多少条
        data: [{}, {}] //详细的数据
    }
});
```

#### buildSql(options)

将当前查询条件生成一个SELECT语句，可以用作子查询的sql语句。

* `options` 操作选项
* `return` promise

```js
D('Cate').where({id: ['>', 10]}).buildSql().then(function(sql){
  //sql = SELECT `id` FROM `meinv_cate` WHERE id > 10
  return D('GROUP').where({cate_id: ['IN', sql, 'exp']});
})
```


#### query(sql, parse)

自定义sql语句进行查询，用于非常复杂的sql语句时使用。

* `sql` 要执行的sql语句
* `parse` 格式参数的数据
* `return` promise

```js
var data = [
  value.field || '*',
  mapOptions.mapfKey,
  value.rTable || self.getRelationTableName(mapOptions.model),
  mapOptions.model.getTableName(),
  whereStr || 'WHERE ',
  value.rfKey || (mapOptions.model.getModelName().toLowerCase() + '_id'),
  mapOptions.model.getPk(),
  value.where ? (' AND ' + value.where) : ''
]
D('Group').query('SELECT b.%s, a.%s FROM %s as a, %s as b %s AND a.%s=b.%s %s', data).then(function(data){
  //查询的数据
})
```

sql语句中支持如下字符串的自动替换：

* `__TABLE__` 替换为当前模型里的表名
* `__USER__` 替换为 `C('db_prefix') + 'user'` 表，其他表类似

```js
//解析后的sql为 SELECT * FROM meinv_group as a LEFT JOIN meinv_user as u ON a.id=u.id WHERE u.id > 10
D('Group').query('SELECT * FROM __TABLE__ as a LEFT JOIN __USER__ as u ON a.id=u.id WHERE u.id > %d', 10);
```

#### execute(sql, parse)

自定义sql语句执行，用户复杂sql语句的情况。

使用方式与`query`相同，只是then里拿到的结果不同。exectue为影响的行数。

#### close()

关闭当前数据库连接，非特殊条件下不要使用该方法。

### 静态方法

#### close()

关闭所有数据库连接。

```js
thinkRequire('Model').close();
```

数据库默认使用长连接的方式，不建议关闭数据库连接。该接口非特殊条件下，不要使用。