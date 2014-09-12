## 命令行模式

thinkjs无缝支持命令行模式的调用，控制器的逻辑可以和普通HTTP请求的逻辑完全一致，可以做到同一个接口即可以HTTP访问，又可以命令行调用。

比如要执行IndexController里的indexAction，可以使用如下的命令：

```shell
node index.js /index/index
```