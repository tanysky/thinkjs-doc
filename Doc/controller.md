## 控制器

控制器是分组下一类功能的集合，每个控制器是一个独立的类文件，每个控制器下有多个操作。

### 定义控制器

创建文件`App/Lib/Controller/Home/ArticleController.js`，表示Home分组下有名为Article的控制器。文件内容如下：

```js
//控制器文件定义
module.exports = Controller(function(){
    return {
        indexAction: function(){
            return this.diplay();
        },
        listAction: function(){
            return this.display();
        },
        detailAction: function(){
            var doc = this.get("doc");
        }
    }
});
```

控制器的名称采用驼峰法命名，且首字母大写。

控制器类必须继承于Controller或者Controller的子类，建议每个分组下都有个`BaseController`，其他的Controller继承该分组下的BaseController，如：

```js
//继承Home分组下的BaseController
module.exports = Controller("Home/BaseController", function(){
    return {
        indexAction: function(){

        }
    }
})
```

`注意：` 这里的`Home/BaseController`不能只写成`BaseController`，那样会导致BaseController文件找不到，必须要带上分组名称。


### 初始化方法

js本身并没有在一个类实例化时自动调用某个方法，但thinkjs实现一套自动调用的机制。自动调用的方法名为`init`，如果类有init方法，那么这个类在实例化时会自动调用init方法。

为实现init方法调用的机制，thinkjs里所有的类都是通过 [Class](/doc/api_global.html#class) 函数创建。

如果控制器里要重写init方法，那么必须调用父类的init方法，如：
```js
module.exports = Controller(function(){
    return {
        init: function(http){ //这里会传递一个包装后的http对象进来
            this.super("init", http); //调用父级的init方法，并将http作为参数传递过去
            //额外的逻辑
        }
    }
})
```

这里说个技巧：一般系统后台都需要用户登录才能访问，但判断用户是否登录一般都是异步的，不可能在每个Action里都去判断是否已经登录。

这时候就可以在init方法里判断是否已经登录，并且把这个promise返回，后续的action执行则是在这个then之后执行。如：

```
module.exports = Controller(function(){
    return {
        init: function(http){ //这里会传递一个包装后的http对象进来
            this.super("init", http); //调用父级的init方法，并将http作为参数传递过去
            //login请求不判断当前是否已经登录
            if(http.action === 'login'){
                return;
            }
            //通过获取session判断是否已经登录
            var self = this;
            return this.session("userInfo").then(function(data){
                if(isEmpty(data)){
                    //如果未登录跳转到登录页。由于redirect方法返回的是个pendding promise，那么后面的action方法并不会被执行
                    return self.redirect("/login"); 
                }else{
                    //设置后，后面的action里直接通过this.userInfo就可以拿到登录用户信息了
                    self.userInfo = data;
                }
            })
        }
    }
})
```

### 前置和后置操作

thinkjs支持前置和后置操作，默认的方法名为`__before`和`__after`，可以通过如下的配置修改：

```js
//前置、后置配置名称
"before_action_name": "__before",
"after_action_name": "__after"
```

调用`__before`和`__after`方法时，会将当前请求的action传递过去。

```js
//前置和后置操作
module.exports = Controller(function(){
    return {
        __before: function(action){
            console.log(action)
        },
        __after: function(action){
            console.log(action);
        }
    }
})
```

`__before`，`action`， `__after`是通过promise的链式调用的，如果当前操作返回一个reject promise或者pedding promise，那么则会阻止后面的执行。

### Action参数自动绑定

请求参数的值，一般是在action里通过get(name)或者post(name)来获取，如：

```js
var name = this.get("name");
var value = this.post("value");
```

为了获取方便，thinkjs里提供了一种方便的获取参数的方式，将参数绑定到Action上，需要开启如下的配置：

```js
'url_params_bind': true //该功能默认开启
```

开启后，就可以通过下面的方式来获取参数

```js
module.exports = Controller(function(){
    return {
        detailAction: function(id){
            //参数绑定后，这里的参数id值即为传递过来的id值
            //如：访问 /article/10 的话，这里的id值为10
            //这里可以是多个参数
        }
    }
})
```

<div class="alert alert-warning">
    `注意`：参数绑定是通过将函数 toString 后解析形参字符串得到的，如果代码上线时将js压缩的话那么就不能使用该功能了。
</div>

### 空操作

空操作是指系统在找不到请求的操作方法的时候，会定位到空操作(`__call`)方法执行，利用这个机制，可以实现错误页面和一些url的优化。

默认空操作对应的方法名为`__call`，可以通过下面的配置修改：

```js
//空操作对应的方法
'call_method': '__call'
```

```js
//空操作方法
module.exports = Controller(function(){
    return {
        __call: function(action){
            console.log(action);
        }
    }
})
```

如果控制器下没有空操作对应的方法，那么访问一个不存在的url时则会报错。

### 空控制器

空控制器是指系统找不到请求的控制器名称的时候，系统会尝试定位到另一个配置的控制器上。配置如下：

```js
//空控制器
//表示当访问一个不存在的控制器时，会执行Home分组下IndexController下的_404Action方法
//如果指定的控制器或者方法不存在，则会报错
call_controller: "Home:Index:_404"
```


### 常用方法

这里列举一些常用的方法，详细的可以去 [Api](/doc/api_controller.html) 文档里查看。

* `get(key)` 获取get参数值
* `post(key)` 获取post参数值
* `file(key)` 获取file参数值
* `isGet()` 当前是否是get请求
* `isPost()` 当前是否是post请求
* `isAjax()` 是否是ajax请求
* `ip()` 获取请求用户的ip
* `redirect(url)` 跳转到一个url，返回一个pedding promise阻止后面的逻辑继续执行
* `echo(data)` 输出数据，会自动调用JSON.stringify
* `end(data)` 结束当前的http请求
* `json(data)` 输出json数据，自动发送json相关的Content-Type
* `jsonp(data)` 输出jsonp数据，请求参数名默认为`callback`
* `success(data)` 输出一个正常的json数据，数据格式为`{errno: 0, errmsg: "", data: data}`，返回一个pedding promise阻止后续继续执行
* `error(errno, errmsg, data)` 输出一个错误的json数据，数据格式为`{errno: errno_value, errmsg: string, data: data}`，返回一个pedding promise阻止后续继续执行
* `download(file)` 下载文件
* `assign(name, value)` 设置模版变量
* `display()` 输出一个模版，返回一个promise
* `fetch()` 渲染模版并获取内容，返回一个prmose，内容需要在promise then里获取
* `cookie(name, value)` 获取或者设置cookie
* `session(name, value)` 获取或者设置session
* `header(name, value)` 获取或者设置header
* `action(name, data)` 调用其他Controller的方法，可以跨分组


### 一些技巧

#### 将Promise return

Acttion里一般会从多个地方拉取数据，如：从数据库中查询数据，这些接口一般都是异步的，并且包装成了Promise。

我们知道，Promise会通过try{}catch(e){}来捕获异常，然后传递到catch里。如：

```js
indexAction: function(){
    D('User').page(1).then(function(data){

    }).catch(function(error){
        //error为异常信息
        console.log(error)
    })
}
```

如果不加catch，那么出错后，我们将无法看到异常信息，这对调试是非常不方便的。并且有时候会从多个地方拉取数据，每个都加catch也极为不便。

为此，thinkjs会自动捕获异常，并打印到控制台。但需要在action将Promise返回，如：

```js
indexAction: function(){
    //这里将Promise return后，如果有异常会打印到控制台里
    return D('User').page(1).then(function(){
        
    })
}
```

#### var self = this

js中，函数作为一等公民存在。这样便利的同时，也会导致函数嵌套非常严重，