## Session

Node.js本身并没有提供Session的功能，但一般网站都有用户登录的功能，为了方便开发者使用，thinkjs提供一套session的机制。

<div class="alert alert-info">
    Session都需要依赖浏览器端的一个Cookie来实现，然后把这个Cookie值作为key到对应的地方去查询，如果有相应的数值表示已经登录，否则表示没有登录。
</div>

### 配置

Session有如下的配置：

```js
//Session配置
session_name: "thinkjs", //session对应的cookie名称
session_type: "File", //session存储类型, 空为内存，还可以为Db
session_path: "", //File类型下文件存储位置，默认为系统的tmp目录
session_options: {}, //session对应的cookie选项
session_sign: "", //session对应的cookie使用签名，如果使用签名，这里填对应的签名字符串
session_timeout: 24 * 3600, //session失效时间，单位：秒
```

默认的Session的存储方式是File类型，可以修改为内存或者Db的方式，对应的值为`""`和`Db`。

如果使用`cluster`模式，则不能使用内存的方式。

使用Db（Mysql）来存储Session需要建如下的数据表：

```
DROP TABLE IF EXISTS `think_session`;
CREATE TABLE `think_session` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `key` varchar(255) NOT NULL DEFAULT '',
  `data` text,
  `expire` bigint(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `cookie` (`key`),
  KEY `expire` (`expire`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

将数据表名`think_session`改为项目里对应的前缀+session。

### 使用Session
在Controller里，可以通过下面的方式使用Session：

```js
//使用Session
//
//获取session里userInfo的值
this.session("userInfo").then(function(data){
    
})
//设置Session
this.session("userInfo", {name: "welefen", "email": "welefen@gmail.com"}).then(function(){

})
//删除Session
this.session().then(function(){

});
```

Session的读、写、删除操作都是异步的。

### 过期Session清除策略

一个用户登录后如果长期不再登录，那么这个Session对应的缓存数据就可以删除了。如果不删除那么会导致数据量越来越大，影响性能。

thinkjs提供了一套定时删除过期Session的策略，由于Session类继承自Cache，所以Session和Cache的清除策略是一样的。 可以通过如下的参数来清除：

```js
//过期数据清除策略
cache_gc_hour: [4], //缓存清除的时间点，数据为小时。
```

这里表示在凌晨4点的时候进行一次清除，你可以修改多个时间点进行清除。如：早上3点、早上9点、下午5点、晚上10点，那么配置值为 `[3, 9, 17, 22]`。

### 示例

判断用户是否登录：

```js
//判断用户是否登录
var self = this;
return this.session("userInfo").then(function(data){
    if(isEmpty(data)){
        //未登录情况跳转到登录页
        return self.redirect("/login")
    }else{
        self.userInfo = data;
        //将用户信息赋值到模版变量里，供模版里使用
        self.assign("userInfo", data);
    }
})
```

用户登录成功写入Session：
```js
//用户登录成功写入Session
var name = this.post("name"); //获取post过来的用户名
var pwd = this.post("pwd"); //获取post过来的密码
var self = this;
return D('User').where({ //根据用户名和密码查询符合条件的数据
    name: name,
    pwd: md5(pwd)
}).find().then(function(data){
    if(isEmpty(data)){
        //用户名或者密码不正确，返回错误信息
        return self.error(403, "用户名或者密码不正确");
    }else{
        return self.session("userInfo", data);
    }
}).then(function(){
    //返回正确信息
    return self.success();
})
```

