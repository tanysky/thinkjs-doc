## WebSocket

thinkjs可以无缝的支持`websocket`，写WebSocket的逻辑和普通的Http请求的方式一致，底层使用了[websocket](https://www.npmjs.org/package/websocket)模块。

websocket的功能默认是关闭的，使用WebSocket需要开启下面的配置：

```js
//WebSocket配置
"use_websocket": false, //是否使用websocket
```

还有下面的配置可以设置：

```js
//WebSocket配置
websocket_allow_origin: "", //允许从那里发送过来的websocket，可以是字符串、数组、回调函数，为空表示不检测。如："www.welefen.com"
websocket_sub_protocal: "", //websocket子协议，可以是个字符串也可以是回调函数
websocket_message_handle: undefined, //websocket消息处理函数
```

### 数据格式

为了更好的规范浏览器端和服务端之间传送的数据格式，thinkjs默认使用 [jsonrpc 2.0](http://www.jsonrpc.org/specification) 的规范，也可以根据项目数据更改传输的数据格式。

浏览器传送到服务端的数据格式为：

```js
//浏览器发送给服务端的数据格式
var data = {
    jsonrpc: "2.0",
    method: "/test/websocket/message",
    params: {userAgent: navigator.userAgent},
    id: 1
}
```

服务端发送给浏览器端的数据格式为：

```js
var data = {
    jsonrpc: "2.0",
    id: 1,
    result: {name: "welefen"}
}
```

### 处理逻辑

#### 建立连接

浏览器端可以通过下面的方式创建一个websocket连接。

```js
//websocket连接
var socket = new WebSocket("ws://" + location.hostname + ":1234");
socket.onopen = function(){}
```

这种方式只是创建了个websocket连接，不会调用控制器上任何操作。如果要调用控制器上对应的操作，可以用下面的方式：

```js
//websocket连接
var socket = new WebSocket("ws://" + location.hostname + ":1234/websocket/open");
socket.onopen = function(){}
```

这里会根据路由解析url`/websocket/open`，如：解析到的group为home，controller为websocket, action为open。那么则会执行`App/Lib/Controller/Home/WebsocketController.js`下的`openAction`方法，并且传递进去的http对象上多了如下的属性：

```js
openAction: function(){
    var websocket = this.http.websocket; //通过这个属性可以取到WebSocket对象，如：可以将这个对象存在一个对象池里，方面后续使用（比如：广播事件）
}
```

<div class="alert alert-info">
    WebSocket建立连接时还是HTTP协议，所以在Action里可以取到cookie，也可以设置cookie。
</div>

#### 消息处理

建立连接后，浏览器和服务端就可以双向传输数据了。如：

```js
function getWebSocket(){
    var socket = new WebSocket("ws://" + location.hostname + ":1234/websocket/open");
    var deferred = $.Deferred();
    socket.onopen = function(event) {
        deferred.resolve(socket);
        socket.onmessage = function(event) {
            console.log(JSON.parse(event.data));
        }; 
        socket.onclose = function(event) { 
            socket = null;
        };
    };
    return deferred;
}
getWebSocket().then(function(ws){
    //建立连接后，给服务端发送jsonrpc 2.0格式的数据
    ws.send(JSON.stringify({
        jsonrpc: "2.0",
        method: "/websocket/message",
        params: {name: "welefen"},
        id: 1
    }))
});
```

这里传输的method为`/websocket/message`，表示对应的url为`/websocket/message`，假如根据路由解析后的分组为home，控制器为websocket，操作为message，那么则会执行`App/Lib/Controller/WebscoketController.js`下的`messageAction`方法。

`params`参数值会作为请求参数传递进去，控制器里可以通过`this.get("name")`来获取对应的值。

处理传递请求参数，如果还想传递`headers`信息，那么params格式为：

```js
// params值
params: {
    //请求的headers
    headers: {
        userAgent: "xxx",
        xxx: "yyy"
    },
    //请求参数
    data: {
        name: "welefen"
    }
}
```

和`openAction`方法一样，`messageAction`里也可以从`http`对象上获取websocket对象。

服务端可以通过`echo`方法向浏览器发送数据，如：

```js
//action里发送数据到浏览器
messageAction: function(){
    var data = this.get(); //获取所有传递过来的参数
    this.echo(data); //输出数据到浏览器，框架会自动JSON.stringify
}
```

可以通过`end`方法来关闭websocket连接。

#### websocket关闭

如果浏览器端将websocket关闭了，服务端是可以捕获到这个关闭事件的。捕获需要在`openAction`里进行。

```js
openAction: function(){
    //监听websocket关闭事件
    this.http.on("websocket.close", function(){
        //websocket关闭后逻辑处理
    })
}
```

### 选择子协议

如果项目非常复杂，需要支持不同的数据格式，这时候可以使用子协议的功能。 如：

```js
//websocket连接
var socket = new WebSocket("ws://" + location.hostname + ":1234/websocket/open", ["json", "soap"]);
socket.onopen = function(){}
```

这里表示浏览器端支持json和soap协议（如果是一个值可以是个字符串），服务端返回时需要选择一个协议告知浏览器。

服务端可以通过下面的配置来指定用哪个子协议，如：

```js
//服务端子协议配置
"websocket_sub_protocal": "soap" //这里表示服务端使用soap协议
```

也可以配置一个回调函数，会将浏览器支持的子协议列表作为参数传递进去。如：

```js
//服务端子协议配置
"websocket_sub_protocal": function(protocals){
    return protocals[0]; //选择第一个子协议
}
```

`注意：` 如果服务端返回的子协议不在浏览器传递过去的列表里，则会报错。

### 自定义数据格式

thinkjs默认使用`jsonrpc 2.0`的数据格式来传输，如果这种数据格式不能满足项目的需要，那么可以根据项目特点自定义数据格式。

自定义数据格式后，需要在项目里实现数据解析和发送逻辑。

#### 数据解析与发送

自定义数据格式后，需要定义下面的配置来实现数据的解析和发送。如：

```js
"websocket_message_handle": function(data, connection, app, type){
    //data为浏览器传递过来的数据
    //connection为websocket的连接句柄
    //app为系统的App对象
    //type为数据格式，一种是字符串，一种是二进制数据
}
```

逻辑中必须实现如下的逻辑：

```js
var pars = {
    host: "", //请求host
    url: url, //请求的url，从data某个属性读取
    headers: headers, //headers, 从data某个属性读取
    //发送数据
    write: function(data, encoding, errMsg){
        connection.send(JSON.stringify(data));
    },
    //关闭连接
    end: function(data){
        if (data) {this.write(data)}
        connection.close();
    }
}
//下面几行代码可以直接拷贝，不用修改
var defaultHttp = thinkHttp.getDefaultHttp(pars);
httpInstance = thinkHttp(defaultHttp.req, defaultHttp.res);
//将websocket实例添加到http对象上
httpInstance.http.websocket = connection.socket;
httpInstance.run(app.listener);
```

其中pars里的`url`, `write`, `end`必须要实现，否则会报错。这里使用`write`和`end`，而不是`send`和`close`是为了和HttpResponse对象的方法名相同。

#### 广播数据发送逻辑

广播数据发送是指在一个websocket请求里向其他所有或者部分的websocket发送数据，需要在`openAction`里定义`websocket.send`方法。如：

```
openAction: function(){
    var websocket = this.http.websocket;
    websocket.send = function(data){
        //调用websocket.connection.send方法直接发送
        websocket.connection.send(JSON.stringify(data));
    }
}
```

### websocket id

为了后续处理方便，系统会在websocket对象上加上id属性，属性值是单调增的，保证每个websocket对象的id值都不一样。

```js
//获取websocket的id
openAction: function(){
    var id = this.http.websocket.id;
},
//message里也能获取websocket的id
messageAction: function(){
    var id = this.http.websocket.id;
}
```

### 超时处理

有时候有些websocket会一直连接，但没有任何数据交互（比如：一些用来攻击连接的websocket）。如果不把这些websocket清理掉，那么占用的内容一直无法释放，同时对广播事件的性能也有影响。

thinkjs会每个websocket都添加了`activeTime`属性，这个属性值在每次有数据传输时都会更新。

有了这个时间点，那么就可以在控制器里里加上超时处理的逻辑了。比如：三十分钟清理一次

```
var websocketList = {};
//30分钟执行一次清理操作
setInterval(function(){
    var now = Date.now();
    for(var id in websocketList){
        var websocket = websocketList[id];
        if((now - websocket.activeTime) >= 30 * 60 * 1000){
            //超时后关闭websocket
            websocket.close();
            //从列表里清除
            delete websocketList[id];
        }
    }
}, 30 * 60 * 1000);

module.exports = Controller(function(){
    openAction: function(){
        var websocket = this.http.websocket;
        //将当前的websocket加到列表里
        websocketList[websocket.id] = websocket;
    }
})
```

