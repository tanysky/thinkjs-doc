## 快速入门

thinkjs需要Node.js的版本`>=0.10.x`，可以通过`node -v`命令查看当前node的版本。如果未安装node或者版本过低，请到 [Node.js](http://nodejs.org/) 官网进行安装或升级。

使用thinkjs时，假设你已经有了Node.js开发相关的经验。


### 安装thinkjs

安装thinkjs非常简单，通过如下的命令即可安装：

```shell
sudo npm install -g thinkjs-cmd
```

如果安装失败，可能是npm服务异常或者是被墙了，可以使用国内的 [cnpm](http://cnpmjs.org/) 服务进行安装。如：

```shell
sudo npm install -g thinkjs --registry=http://r.cnpmjs.org
```

安装完成后，可以通过下面的命令查看thinkjs的版本号：

```js
thinkjs -v
```

### 新建项目

```shell
# 在合适的位置创建一个新目录，new_dir_name为你想创建的文件夹名字
mkdir new_dir_name; 
# 进入这个目录
cd new_dir_name;
# 通过thinkjs命令创建项目
thinkjs .
```

执行后，如果当前环境有浏览器，会自动用浏览器打开 <http://127.0.0.1:8360>，并且会看到如下的内容：

```
hello, thinkjs!
```

看到这个内容后，说明项目已经成功创建。


### 手动启动项目

创建项目时，会自动通过子进程来启动node服务。为了后续开发方便，最好还是手动来启动。

通过键盘操作`ctrl+c`结束当前的进程，cd到`www`目录下，执行`node index.js`来启动服务。


### 项目结构说明
创建项目后，会生成如下的目录结构：

```
├── App
│   ├── Common
│   │   └── common.js    ---- 通用函数文件，一般将项目里的一些全局函数放在这里
│   ├── Conf
│   │   └── config.js    ---- 项目配置文件
│   ├── Lib
│   │   ├── Behavior     ---- 行为类存放位置
│   │   ├── Controller
│   │   │   └── Home
│   │   │       └── IndexController.js   ---- 逻辑控制类
│   │   └── Model        ---- 模型类
│   ├── Runtime          ---- 运行时的一些文件
│   │   ├── Cache        ---- 缓存目录
│   │   ├── Data         ---- 数据目录
│   │   ├── Log
│   │   └── Temp
│   └── View
│       └── Home
│           └── index_index.html      ---- 模版文件，默认使用ejs模版引擎
├── ctrl.sh              ---- 项目启动、停止脚本
└── www
    ├── index.js         ---- 入口文件
    └── resource         ---- 静态资源目录
        ├── css          ---- css文件
        ├── img          ---- 图片文件
        ├── js           ---- js文件
        ├── module       ---- 第三方的一些组件
        └── swf          ---- flash文件
```


### 文件说明

下面对几个重要的文件进行简单的说明。

#### 入口文件

`www/index.js`

```js
//定义APP的根目录
global.APP_PATH = __dirname + "/../App";
//静态资源根目录
global.RESOURCE_PATH = __dirname;
global.ROOT_PATH = __dirname;
global.APP_DEBUG = true; //是否开启DEBUG模式
require('thinkjs');
```

默认开启debug模式，该模式下文件修改后立即生效，不必重启node服务。

<div class="alert alert-warning">
    线上环境切记要将debug模式关闭，即：APP_DEBUG=false
</div>

debug模式详细说明请见 [调试](/doc/debug.html) 里相关内容。

#### 配置文件

`App/Conf/config.js`

```js
module.exports = {
  //配置项: 配置值
  port: 8360, //监听的端口
  db_type: 'mysql', // 数据库类型
  db_host: 'localhost', // 服务器地址
  db_port: '', // 端口
  db_name: '', // 数据库名
  db_user: 'root', // 用户名
  db_pwd: '', // 密码
  db_prefix: 'think_', // 数据库表前缀
};
```

可以在配置文件中修改框架默认的配置值，如：将http监听的端口号由默认的8360改为1234，那么这里加上 `"port": 1234`，重启服务后就生效了(ps: 要把url中的端口号改为1234才能正常访问哦)。

框架默认的配置值请见 [附录 - 默认配置](/doc/appendix#appendix_config)

#### 函数文件

`App/Common/common.js`

该文件下定义一些当前应用常用的函数，可以直接放在global下，该文件在系统启动时自动加载。

```js
global.getDate = function(){
    return 'xxx';
};
global.getSliceUrl = function(url, length){
    return '';
}
```

这些函数在其他地方可以直接使用，无需在require。


#### 控制器文件

`App/Lib/Controller/Home/IndexController.js`

```js
/**
 * controller
 * @return 
 */
module.exports = Controller(function(){
    "use strict";
    return {
        indexAction: function(){
            //render View/Home/index_index.html file
            this.display(); 
        }
    };
});
```

该文件为一个基础的控制器文件，只有一个indexAction，这个action直接渲染`View/Home/index_index.html`模版文件。

除了渲染文件，你可以直接输出字符串。可以将这里改为`this.end('hello word')`，刷新浏览器后，显示为hello word。

控制器详细内容请见 [控制器](/doc/controller.html) 相关内容。


#### 启动服务shell

默认情况下，需要`cd www`目录下，执行`node index.js`来启动服务，这种方式在开发环境下比较方便。

但如果线上环境还是这么启动的话就比较麻烦了。


为了方便开发者在线上环境启动/关闭服务，系统提供了`ctrl.sh`脚本（该文件与www目录平级）。可以通过如下的命令操作：

* `sh ctrl.sh start` 启动服务
* `sh ctrl.sh stop` 关闭服务
* `sh ctrl.sh restart` 重启服务