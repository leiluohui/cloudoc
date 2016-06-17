# cloudark 的文档

cloudark 是一套开发分布式高并发应用的平台以及工具。他包含了如下内容：

## cloudeer

这是微服务的注册和管理工具。

## cloudoll

包含了一系列工具，协助快速建立应用程序。

## cloudarling

这是一个微服务，提供 passport 服务。以及权限管理等操作。


## 从 0 开始创建一个微服务

### 运行 cloudeer

从 git 上 下载源码：

```
git clone https://code.aliyun.com/cloudark/cloudeer.git
```

进入到目录，进行 node 前戏工作。

```
npm install
```

手工创建一个 ./data 的目录用来存储数据

运行：

```
node index.js
```

访问 http://localhost:8801/view

无需改动 cloudeer 任何代码。

打完收工!

### 使用 cloudoll 创建一个微服务

创建一个 node 应用程序 （怎么创建？自己去研究咯）。

引入 cloudoll 包

```
npm i cloudoll --save
```

创建一个文件： /config/development.js 内容如下：

```
module.exports = {
  debug         : true,
  port          : 22221,
  cloudeer_url  : "http://127.0.0.1:8801",
  my_ip         : "127.0.0.1",
  controller_dirs: ['/api/open', '/api/admin'],
  schema_path    : './schema',
  my_errors_path  : './my-errors.js'
};
```

创建一个入口文件 index.js

```
var KoaApp = require('cloudoll').KoaApplication;
var app    = KoaApp();
var router = app.router;
var cloudeer = app.cloudeer;
```

上面的两个变量  router 和 cloudeer 留着待用。

创建文件 /api/open/hello.js

```
module.exports = {
  world: function () {
    this.body = "你好世界。";
  }
};

```

请使 cloudeer 服务运行先。

现在启动服务：

```
node index.js
```


现在访问一下试试

http://localhost:22221/open/hello/world

稍等一下访问

http://localhost:8801/view

和

http://localhost:8801/methods

试试。

好像很简单吧！

万里长征才走完第一步。

# cloudoll has more...

cloudoll 包含更多的功能，其中大多数功能都可以单独拿出来独立使用：

* [内置的 KoaApplication (上面的例子用的是这个)](./KoaApplication.md)

* 错误/例外处理

* 自动路由

* 数据的快捷访问 mongo mysql postgres

* 包含数据访问的服务层和控制层的基类（对应传统的MVC）

* schema 的自动验证

* 作为消费者调用远程其他微服务

* 权限验证（路由级别）

* 内网接口的防火墙功能（正在实现）


