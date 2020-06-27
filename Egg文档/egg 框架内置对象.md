[toc]

egg 框架内置对象

# 1. Application
## 1.1 定义
Application 是全局应用对象, 它继承自Koa.Application 一个项目中只会实例化一个Application 对象;

我们可以在 Application 对象上挂载一些全局的方法和对象(联想浏览器的window 和Node 的global)

## 1.2 事件
1. server: 该事件在一个worker 中只会触发一次, 在 HTTP 服务完成启动后，会将 HTTP server 通过这个事件暴露出来给开发者.
2. error: 在运行过程中, 只要有任何异常均会触发该事件, 一般用于自定义的日志上报处理
3. request/response: 在就收请求和响应时会触发该事件, 并会暴露请求的上下文, 开发者可以监听这两个事件来进行日志记录

```
module.exports = app => {
  app.once('server', server => {
    // websocket
  });
  app.on('error', (err, ctx) => {
    // report error
  });
  app.on('request', ctx => {
    // log receive request
  });
  app.on('response', ctx => {
    // ctx.starttime is set by framework
    const used = Date.now() - ctx.starttime;
    // log total cost
  });
};
```

## 1.3 获取方式
### 1. 启动自定义脚本: 参数的传递
```
// app.js
module.exports = app => {
  app.cache = new Cache();
};
```

### 2. 继承于 Controller, Service 基类的实例中: 通过this.app 访问
```
class UserController extends Controller {
  async fetch() {
    this.ctx.body = this.app.cache.get(this.ctx.query.id);
  }
}
```

### 3. 在context 对象上: 通过this.ctx.app 访问
```
class UserController extends Controller {
  async fetch() {
    this.ctx.body = this.app.cache.get(this.ctx.query.id);
  }
};
```

# 2. Context
## 2.1 定义
Context 是一个请求级别的对象，继承自 Koa.Context。在每一次收到用户请求时，框架会实例化一个 Context 对象，这个对象封装了这次用户请求的信息, 并提供了许多便捷的方法来获取请求参数或者设置响应信息

框架会将所有的 Service 挂载到 Context 实例上

## 2.2 获取的方式
### 1.在 Middleware, Controller 以及 Service 中, 通过this.ctx 获取
```
class HomeController extends Controller {
  async index() {
    // this.ctx.body 表示设置返回的数据内容, 可以理解为定义一个接口内容
    this.ctx.body = 'Hello world';
  }
}
```

### 2. 在Middleware 中, 通过参数获取
```
// Koa v1
function* middleware(next) {
  // this is instance of Context
  console.log(this.query);
  yield next;
}

// Koa v2
async function middleware(ctx, next) {
  // ctx is instance of Context
  console.log(ctx.query);
}
```

### 3. 通过创建一个匿名Context 对象获取
```
// app.js
module.exports = app => {
  app.beforeStart(async () => {
    const ctx = app.createAnonymousContext();
    // preload before app start
    await ctx.service.posts.load();
  });
}
```

### 4. 在定时器任务中, 通过参数获取
```
// app/schedule/refresh.js
exports.task = async ctx => {
  await ctx.service.posts.refresh();
};
```

# 3. Request & Response
## 3.1 定义
Request 是一个请求级别的对象，继承自 Koa.Request。封装了 Node.js 原生的 HTTP Request 对象，提供了一系列辅助方法获取 HTTP 请求常用参数。

Response 是一个请求级别的对象，继承自 Koa.Response。封装了 Node.js 原生的 HTTP Response 对象，提供了一系列辅助方法设置 HTTP 响应。

## 3.2 获取方式
### 1. 通过Context 对象获取
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.request.query.id;
    ctx.response.body = app.cache.get(id);
  }
}
```

## 3.3 注意点
1. Koa 会在 Context 上代理一部分 Request 和 Response 上的方法和属性
2. ctx.request.query.id 和 ctx.query.id 是等价的，ctx.response.body 和 ctx.body 是等价的
3. 需要注意的是，获取 POST 的 body 应该使用 ctx.request.body，而不是 ctx.body

# 4. Controller
## 4.1 定义
框架提供了一个 Controller 基类，并推荐所有的 Controller 都继承于该基类实现

## 4.2 属性
1. ctx - 当前请求的 Context 实例。
2. app - 应用的 Application 实例。
3. config - 应用的配置。
4. service - 应用所有的 service。
5. logger - 为当前 controller 封装的 logger 对象

## 4.3 获取Controller 方式
### 1. 从egg 上获取(推荐)
```
// 从 egg 上获取（推荐）
const Controller = require('egg').Controller;
class UserController extends Controller {
  // implement
}
module.exports = UserController;
```

### 2. 从app 实例中获取
```
// 从 app 实例上获取
module.exports = app => {
  return class UserController extends app.Controller {
    // implement
  };
};
```

# 5. Service
## 5.1 定义
框架提供了一个 Service 基类，并推荐所有的 Service 都继承于该基类实现

## 5.2 访问方式
### 1. 从 egg 上获取（推荐）
```
const Service = require('egg').Service;
class UserService extends Service {
  // implement
}
module.exports = UserService;
```

### 2. 从 app 实例上获取
```
module.exports = app => {
  return class UserService extends app.Service {
    // implement
  };
};
```

# 6. Helper
## 6.1 定义
Helper 用来提供一些实用的 utility 函数。它的作用在于我们可以将一些常用的动作抽离在 helper.js 里面成为一个独立的函数，这样可以用 JavaScript 来写复杂的逻辑，避免逻辑分散各处

Helper 自身是一个类，有和 Controller 基类一样的属性，它也会在每次请求时进行实例化，因此 Helper 上的所有函数也能获取到当前请求相关的上下文信息

PS: 感觉这个内容可以理解为全局处理函数, 比如深拷贝, 对象比较, 这类函数可以直接定义在这里

## 6.2 获取方式
### 1. 在Context 对象中获取
```
// app/controller/user.js
class UserController extends Controller {
  async fetch() {
    const { app, ctx } = this;
    const id = ctx.query.id;
    const user = app.cache.get(id);
    ctx.body = ctx.helper.formatUser(user);
  }
}
```

### 2. 在模板中获取
```
// app/view/home.nj
{{ helper.shtml(value) }}
```

## 6.3 自定义helper 方法
```
// app/extend/helper.js
module.exports = {
  formatUser(user) {
    return only(user, [ 'name', 'phone' ]);
  }
};
```

# 7. Config
## 7.1 定义
表示整个项目的文件配置内容

## 7.2 获取方式
### 1. 通过app.config 获取

### 2. 在 Controller, Service, Helper 的实例上通过 this.config 获取到 config 对象。

# 8. Logger
## 8.1 定义
日志功能, 用于打印各种级别的日志到对应的日志文件中

## 8.2 分类
1. app.logger: 表示应用级别的日志记录, 用于记录一些业务上与请求无关的信息, 如启动阶段的数据信息
2. app.coreLogger: 一般我们在开发应用时都不应该通过 CoreLogger 打印日志，而框架和插件则需要通过它来打印应用级别的日志，这样可以更清晰的区分应用和框架打印的日志，通过 CoreLogger 打印的日志会放到和 Logger 不同的文件
3. ctx.logger: 打印的日志都会在前面带上一些当前请求相关的信息（如 [$userId/$ip/$traceId/${cost}ms $method $url]），通过这些信息，我们可以从日志快速定位请求，并串联一次请求中的所有的日志
4. ctx.coreLogger: 和 Context Logger 的区别是一般只有插件和框架会通过它来记录日志
5. this.logger: 我们可以在 Controller 和 Service 实例上通过 this.logger 获取到它们，它们本质上就是一个 Context Logger，不过在打印日志的时候还会额外的加上文件路径，方便定位日志的打印位置。

## 8.3 方法
下面方法的意义根据关键字理解即可
1. logger.debug()
2. logger.info()
3. logger.warn()
4. logger.error()

# 9. Subscription
## 9.1 定义
订阅模型是一种比较常见的开发模式，譬如消息中间件的消费者或调度任务。因此我们提供了 Subscription 基类来规范化这个模式

## 9.2 使用
插件开发者可以根据自己的需求基于它定制订阅规范，如定时任务就是使用这种规范实现的。
```
const Subscription = require('egg').Subscription;

class Schedule extends Subscription {
  // 需要实现此方法
  // subscribe 可以为 async function 或 generator function
  async subscribe() {}
}
```