## 指导规范

### 全局函数

全局函数都定义在`App/Common/common.js`文件中，函数名为驼峰式命名。如：

```js
//全局函数定义
global.getPicModel = function(groupId){
    var model = D('Model');
    //extra code
}
```

这样函数`getPicModel`在控制器里就可以直接使用了。

### 类文件

所有的类文件都通过函数`Class`来创建，没有特殊情况，直接赋值给module.exports。如：

```
//require模块放在module.exports前面
var marked = require("markded");
var toc = require("marked-toc");

module.exports = Class(function(){
    //类里面用到的一些变量放在这里，最好不要放在Class之外
    var keyList = {

    }
    return {
        init: function(){

        }
    }
});
```

如果创建的类还有一些属性或者方法，那么可以重新定义一个变量，如：

```
var App = module.exports = Class(function(){ ... })
//listener方法
App.listener = function(){

}
```

thinkjs里包装了很多功能的基类，如：`Model`, `Db`, `Controller`，需要开发这些功能时，可以直接继承这些基类。

### 异步

thinkjs是基于es6-promise来实现的，大大简化了异步回调的代码逻辑。

如果你的项目比较复杂，需要开发一些独立的模块，建议也使用promise的方式。
