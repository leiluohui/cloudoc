# KoaApplication

自动的 koajs 应用程序，只需简单的配置，就可以得到一个强大的『配置优先』，但『约定优于配置』Web 应用程序。

## 配置文件总览

先看看配置文件 /config/develepment.js

```javascript
module.exports = {
  debug                : true,  //可以输出debug信息，在正式部署环境中，请删除此行此配置
  app_name             : "cloudoll", //微服务的名称，在消费端的 rpc 的调用中，这个名字是区分微服务的关键，如果不指定，则读取项目目录名称
  my_host              : "localhost", //本机的 ip，如果不指定，则会读取网卡的第一个 ip 地址
  port                 : 3000, //微服务对外端口
  controller_dirs      : ['/api/open', '/api/admin', '/api/inner'],  //cloudoll 会自动扫描这些目录，并映射出路由
  schema_path          : './schema', //json-schema 的存放路径，可以方便的在应用中自动完成数据校验
  my_errors_path       : './errors.js', //自定义的错误信息路径，按照约定的方式编写，可以得到一致的错误消息。
  koa_middles_forbidden: {
    clouderr_handle: true,  //配置此节点，将禁用统一的错误信息功能。
    auto_router    : true, //将禁用自动路由功能
    json_validator : true, //将禁用 json-schema 的验证功能
    authenticate   : true //将禁用权限验证模块
  },
  cloudeer             : {
    type    : 'tcp', //支持 'rest', 'tcp'
    host    : '112.74.29.211', //注册中心的地址
    port    : 2346, //注册中心的端口
    username: 'knock', //注册中心的用户名
    password: 'password', //注册中心的密码
    not_a_consumer: true //标记本服务不是一个消费端，以节省注册中心和微服务的通讯消耗
  }
};


```

本机地址 my_host 的读取顺序为：

config.my_host, 环境变量 my_host，自动读取第一个网卡的 ipv4 地址。

其实，所有的东西都可以不用配置哦。

系统会按照默认约定正确运行。


## 根据不同的配置文件启动应用

在实际生产环境中，我们需要不同的配置来运行程序。

下面的例子将启动 /config/product.js 配置文件

```javascript
var KoaApplication = require('cloudoll').KoaApplication;
process.env.NODE_ENV = "product";
var app = KoaApplication();
```

在实际应用中，可以把 NODE_ENV 配置到系统环境变量中。比如：

```bash
#!/usr/bin/bash

export NODE_ENV='product'
```

这样就可以不用修改项目里面的代码而加载不同的配置文件。

## 我可以独立使用吗？

可以！如果没有配置 cloudeer 节点，那么应用程序将会变成一个普通的 web 应用程序。

只需要将你的 'controller' 文件放入 '/api/open', '/api/admin', '/api/inner'  这三个文件夹中。 

## KoaApplication 包含如下的一些特性

### app.context 扩展方法

可以在路由方法中使用 this. 直接调用。

#### echo(somthing)

在路由中，可以使用 this.echo("xxxx") 输出结果。

他会自动封装成

```json
{ "errno": 0, "data": "xxxx"}
```

#### *getCloudeer(service, url, params)

可以使用此方法直接调用一个远程微服务的 GET 方法，这是一个 generator。

如果没有配置 cloudeer 节点，那么将不会有这个方法。


#### *postCloudeer(service, url, params)

和 *getCloudeer  一样，调用远程微服务的 POST 方法。



代码示例:

```
module.exports = {
  world: function *() {
    var dist = yield this.getCloudeer("cloudarling", "/open/district/children");
    this.echo(dist);
  }
};
```

#### *getUser()

使用此方法可以直接获取用户的基本信息。在包含 ticket 的情况下。

此方法依赖 [cloudarling](https://github.com/cloudoll/cloudarling) 微服务。


### 自动映射 cloudeer 远程调用


程序自动映射了下面三个路由：


**POST /cloudeer**

参数示例：
```json
{
    "method": "GET",
    "service": "cloudarling",
    "url": "/open/account/login",
    "params": { "passport": "xxx", "password": "xxxxxxx" }
}
```


**POST /cloudeer/get**

参数和上面的原型一样。默认 method 为 GET。

**POST /cloudeer/post**

参数和上面的原型一样。默认 method 为 POST。

~~如果 cookie 里有 ticket ，则可以自动放入参数中，调用的时候无需指定 ticket。~~ 由于存在 csrf 攻击风险，此特性不再支持。 

这个功能无法通过配置文件关闭，除非删除了 cloudeer 配置节点。


### 自动加载错误集合

KoaApplication 自动加载了 cloudoll 的 errors 模块。

这个模块封装了 web 开发的常用的错误类型，方便开发者在程序中调用。

你也可以加载自定义的错误集合，只需要将下面的代码放到合适的位置就会自动加载。

代码示例如下：

```javascript
// 默认放在根目录下： errors.js
// 这个文件的文件名可以配置，配置节点为：
// my_errors_path: './errors.js'
module.exports = {
  CUSTOM00      : {errno: -1, errText: "%s"},
  SYSTEM_ERROR01: {errno: -10, errText: "系统错误"}
}

```

上面的示例中：

errors.CUSTOM00 需要通过方法调用。

```javascript
throw errors.CUSTOM00('我爱xxx');
```

errors.SYSTEM_ERROR01  直接调用，他已经是一个 Clouderr 的实例了。

```javascript
throw errors.SYSTEM_ERROR01;
```

是否需要参数以及参数的个数 是和 %s 的个数相关的。

[更多的错误处理请看这个文档](./Clouderr.md)


### 错误的统一处理中间键

加载了两个中间件：errorsHandleAhead, errorsHandleBehind

可以通过配置文件关闭：

```javascript
koa_middles_forbidden: {
    clouderr_handle: true
}
```

关于错误，具体参考 [这个文档](./Clouderr.md)


### 已经自动加载的中间件：

* koa-bodyparser [参见这里](https://github.com/koajs/bodyparser)

这个中间件自动解析 post 信息。

* queryStringParser 这个插件将 query string 作为 json 放入 this.qs 中。



### 加载额外前置中间键

如果有另外的中间价需要加载，你可以直接使用

```
app.use(...)
```

但如果中间件需要提前加载，比如 『静态文件』 服务，则需要将中间件放入构造函数中，示例如下：

```javascript
var cloudoll = require('cloudoll');

var serve = require('koa-static');
var app   = new cloudoll.KoaApplication({
  middles: [serve('./publish')]
});
```



### 权限验证中间件 authenticate

可以通过配置文件关闭：

```javascript
koa_middles_forbidden: {
    authenticate: true
}
```

使用权限验证插件的前提本服务必须同时是一个消费端（consumer）。

微服务的内部接口（/inner 开头的 url）也同时被验证，只有那些在注册服务注册过的服务器才能访问内部接口。

纯服务的微服务可以标记 not_a_consumer 为 true 。

```javascript
cloudeer: {
    not_a_consumer: true
}
```


### 自动路由

KoaApplication 会自动扫描目录，将方法和 url 自动对应起来，就像「传统的 MVC 那样」。

这样，你就可以不用写那些 router 了。

你可以不用担心自动扫描会影响性能，因为扫描操作只会在程序启动的时候运行一次。

[具体参考这里](./autoRouters.md)

### 自动 json-schema 验证

### 自动提交服务注册

### 自动提交方法列表

### 自动下载服务列表


