## Controller

 此文档表示控制器下接口，在子控制器下可以直接使用。

### ip()

获取当前请求的用户ip。

```js
testAction: function(){
    //用户ip
    var ip = this.ip();
}
```

如果项目部署在本地的话，返回的ip为`127.0.0.1`。

### isGet()

判断当前请求是否是get请求。

```js
testAction: function(){
    //如果是get请求直接渲染模版
    if(this.isGet()){
        return this.display();
    }
}
```

### isPost()

判断当前请求是否是post请求。

```js
testAction: function(){
    //如果是post请求获取对应的数据
    if(this.isPost()){
       var data = this.post();
    }
}
```

### isMethod(method)

判断是否是一个特定类型的请求。

```js
testAction: function(){
    //判断是否是put请求
    var isPut = this.isMethod("put");
    //判断是否是delete请求
    var isDel = this.isMethod("delete");
}
```

http支持的请求类型为：`GET`, `POST`, `PUT`, `DELETE`, `HEAD`, `OPTIONS`, `PATCH`。

### isAjax(method)

判断是否是ajax类型的请求。

```js
testAction: function(){
    //只判断是否是ajax请求
    var isAjax = this.isAjax();
    //判断是否是get类型的ajax请求
    var isGetAjax = this.isAjax("get");
    //判断是否是post类型的ajax请求
    var isPostAjax = this.isAjax("post");
}
```

### isWebSocket()

判断是否是websocket请求。

```js
testAction: function(){
    //是否是WebSocket请求
    var isWS = this.isWebSocket();
}
```

### get(name)

获取get参数值，默认值为空字符串。

```js
testAction: function(){
    //获取name值
    var name = this.get("name");
}
```

如果不传`name`，则为获取所有的参数值。

```js
testAction: function(){
    //获取所有的参数值
    //如：{name: "welefen", "email": "welefen@gmail.com"}
    var data = this.get();
}
```

### post(name)

获取post过来的值，默认值为空字符串。
```js
testAction: function(){
    //获取post值
    var name = this.post("name");
}
```

如果不传`name`，则为获取所有的post值。
```js
testAction: function(){
    //获取所有的post值
    //如：{name: "welefen", "email": "welefen@gmail.com"}
    var data = this.post();
}
```

### param(name)

获取get或者post参数，优先从post里获取。等同于下面的方式：

```js
testAction: function(){
    //这2种方式的结果是一样的
    var name = this.param("name");
    var name = this.post("name") || this.get("name");
}
```

### file(name)

获取上传的文件。

```js
testAction: function(){
    //获取表单名为image上传的文件
    var file = this.file("image");
}
```

如果不传`name`，那么获取所有上传的文件。

@TODO 列出`file`数据格式

### header(name, value)

获取或者发送header信息。

#### 获取头信息

获取单个头信息

```js
//获取单个头信息
testAction: function(){
    var value = this.header("accept");
}
```

获取所有的头信息

```js
//获取所有的头信息
testAction: function(){
    var headers = this.header();
}
```

#### 设置头信息

设置单个头信息

```js
//设置单个头信息
testAction: function(){
    this.header("Content-Type", "text/xml");
}
```

批量设置头信息

```js
testAction: function(){
    this.header({
        "Content-Type": "text/html",
        "X-Power": "test"
    })
}
```

### userAgent()

获取浏览器端传递过来的userAgent。

```js
//获取userAgent
testAction: function(){
    var userAgent = this.userAgent();
}
```

### referer()

获取referer。

```js
//获取referrer
testAction: function(){
    var referrer = this.referer();
}
```

### cookie(name, value, options)

获取或者设置cookie

#### 获取cookie

获取单个cookie

```js
//获取单个cookie
testAction: function(){
    var name = this.cookie("name");
}
```

获取所有cookie

```js
//获取所有cookie
testAction: function(){
    var cookies = this.cookie();
}
```

#### 设置cookie

设置cookie默认会用如下的配置，可以在`App/Conf/config.js`里修改。

```js
//设置cookie默认配置
cookie_domain: "", //cookie有效域名
cookie_path: "/",   //cookie路径
cookie_timeout: 0, //cookie失效时间，0为浏览器关闭，单位：秒
```

```js
//设置cookie
testAction: fucntion(){
    this.cookie("name", "welefen");
    //修改发送cookie的选项
    this.cookie("value", "xxx", {
        domain: "",
        path: "/",
        httponly: true, //httponly
        secure: true, //https下才发送cookie到服务端
        timeout: 1000 //超时时间，单位秒
    })
}
```

### session(name, value)

获取或者设置session。

#### 获取session

```js
//获取session
testAction: function(){
    //获取session是个异步的过程，返回一个promise
    this.session("userInfo").then(function(data){
        if(isEmpty(data)){
            //无用户数据
        }
    })
}
```

#### 设置session

```js
testAction: function(){
    //设置session也是异步操作
    this.session("userInfo", {name: "welefen"}).then(function(){

    })
}
```

#### 删除session

```js
testAction: function(){
    //不传任何参数表示删除session，比如：用户退出的时候执行删除的操作
    this.session().then(function(){

    })
}
```

### redirect(url, code)

url跳转。

* `code` 默认值为302
* `return` 返回一个pendding promise，阻止后面代码继续执行

```js
testAction: function(){
    var self = this;
    return this.session("userInfo").then(function(data){
        if(isEmpty(data)){
            //如果用户未登录，则跳转到登录页面
            return self.rediect("/login");
        }
    })
}
```


### assign(name, value)

给模版变量赋值，或者读取已经赋值的模版变量。

#### 变量赋值

单个变量赋值

```js
testAction: function(){
    //单个变量赋值
    this.assign("name", "value");
}
```

批量赋值

```js
testAction: function(){
    this.assign({
        name: "welefen",
        url: "http://www.thinkjs.org/"
    })
}
```

#### 读取赋值变量

```js
testAction: function(){
    //读取已经赋值的变量
    var value = this.assign("url");
}
```

### fetch(templateFile)

### display(templateFile)

### action(action, data)

可以跨分组、跨控制器的action调用。

* `return` 返回一个promise

```js
testAction: function(){
    //调用相同分组下的控制器为group，操作为detail方法
    var promise = this.action("group:detail", [10]);
    //调用admin分组下的控制器为group，操作为list方法
    var promise = this.action("admin:group:list", [10])
}
```

### jsonp(data)

jsonp数据输出。

会自动发送`Content-Type`，默认值为`application/json`，可以在配置`json_content_type`中修改。

jsonp的callback名称默认从参数名为`callback`中获取，可以在配置`url_callback_name`中修改。

callback名称会自动做安全过滤，只保留`\w\.`字符。

```js
testAction: function(){
    this.jsonp({name: "xxx"});
}
```

假如当前请求为`/test?callback=functioname`，那么输出为`functioname({name: "xxx"})`。

`注`： 如果没有传递callback参数，那么以json格式输出。

### json(data)

json数据输出。

会自动发送`Content-Type`，默认值为`application/json`，可以在配置`json_content_type`中修改。


### status(status)

发送状态码，默认为404。

```js
//发送http状态码
testAction: function(){
    this.status(403);
}
```

### echo(data, encoding)

输出数据，可以指定编码。

默认自动发送`Content-Type`，值为`text/html`。可以在配置`tpl_content_type`中修改，也可以设置`auto_send_content_type = false`来关闭发送`Content-Type`。

* `data` 如果是数组或者对象，自动调用JSON.stringify。如果不是字符串或者Buffer，那么自动转化为字符串。 

```js
testAction: function(){
    this.echo({name: "welefen"});
}
```

### end(data, encoding)

结束当前的http请求数据输出。

如果是通过`this.echo`输出数据，那么在最后必须要调用`this.end`来结束输出。

* `data` 如果data不为空，那么自动调用`this.echo`来输出数据

```js
testAction: function(){
    this.end({name: "welefen"});
}
```

### type(ext)

发送`Content-Type`。

* `ext` 如果是文件扩展名，那么自动查找该扩展名对应的`Mime-Type`。

```js
testAction: function(){
    this.type("text/html");
    this.type("js"); //自动查找js对应的Mime-Type
    this.type("css"); //自动查找css对应的Mime-Type
}
```

### download(file, contentType, filename)

下载文件。

* `file` 要下载的文件路径
* `contentType` 要发送的`Content-Type`,如果没传，自动从文件扩展名里获取
* `filename` 下载的文件名
* `return` 返回一个promise

```js
testAction: function(){
    var file = "/home/welefen/a.txt";
    this.download(file, 'text/html', '1.txt').then(function(){
        //下载完成后可以在这里进行一些操作，如：将下载次数+1
    });
}
```

### success(data)

输出一个错误号为0的数据。

* `return` 返回一个pendding promise，阻止后面继续执行。

```js
testAction: function(){
    return this.success({email: "xxx@gmail.com"})
}
```

浏览器拿到的数据为：

```js
{
    errno: 0,
    errmsg: "",
    data: {
        email: "xxx@gmail.com"
    }
}
```

其中`errno`表示错误号（此时为0），`errmsg`表示错误信息（此时为空）。`data`里存放具体数据。

会自动发送`Content-Type`，默认值为`application/json`，可以在配置`json_content_type`中修改。

其中`errno`和`errmsg`可以通过下面的配置修改：

```js
error_no_key: "errno", //错误号的key
error_msg_key: "errmsg", //错误信息的key
```


### error(errno, errmsg, data)

输出一个errno > 0的信息。

* `errno` 错误号，默认为1000，可以通过配置`error_no_default_value`修改
* `errmsg` 错误信息，字符串
* `data` 额外的数据
* `return` 返回一个pedding promise，阻止后续继续执行

```js
//输出一个错误信息
testAction: function(){
    return this.error(1001, "参数不合法");
}
```

浏览器拿到的数据为：

```js
{
    errno: 1001,
    errmsg: "参数不合法"
}
```

也可以值输出错误信息，那么错误号为配置`error_no_default_value`的值。

```js
testAction: function(){
    return this.error("参数不合法");
}
```

浏览器拿到的数据为：

```js
{
    errno: 1000,
    errmsg: "参数不合法"
}
```

也可以传个对象进去，如：

```js
testAction: function(){
    return this.error({
        errno: 10001,
        errmsg: "参数不合法"
    })
}
```

### filter(value, type)

变量过滤器，具体请见 [这里](/doc/data_filter.html)。

### valid(data, type)

数据校验，具体请见 [这里](/doc/data_valid.html)。
