[toc]

控制器(Controller)

# 1, 定义
一个controller 是可以定义多个内容的

Controller 负责解析用户的输入，处理后返回相应的结果:
1. 在 RESTful 接口中，Controller 接受用户的参数，从数据库中查找内容返回给用户或者将用户的请求更新到数据库中
2. 在 HTML 页面请求中，Controller 根据用户访问不同的 URL，渲染不同的模板得到 HTML 返回给用户
3. 在代理服务器中，Controller 将用户的请求转发到其他服务器上，并将其他服务器的处理结果返回给用户

框架推荐 Controller 层主要对用户的请求参数进行处理（校验、转换），然后调用对应的 service 方法处理业务，得到业务结果后封装并返回:
1. 获取用户通过 HTTP 传递过来的请求参数。
2. 校验、组装参数。
3. 调用 Service 进行业务处理，必要时处理转换 Service 的返回结果，让它适应用户的需求。
4. 通过 HTTP 将结果响应给用户

# 2. 编写Controller
## 2.1 原则
1. 所有的 Controller 文件都必须放在 app/controller 目录下
2. 支持多级目录

## 2.2 编写方式
### 1. Controller 类（推荐）
```
// app/controller/sub/post.js
const Controller = require('egg').Controller;
class PostController extends Controller {
  async create() {
    const { ctx, service } = this;
    const createRule = {
      title: { type: 'string' },
      content: { type: 'string' },
    };
    // 校验参数
    ctx.validate(createRule);
    // 组装参数
    const author = ctx.session.userId;
    const req = Object.assign(ctx.request.body, { author });
    // 调用 Service 进行业务处理
    const res = await service.post.create(req);
    // 设置响应内容和响应状态码
    ctx.body = { id: res.id };
    ctx.status = 201;
  }
}
module.exports = PostController;

// app/router.js
module.exports = app => {
  app.router.post('createPost', '/api/posts', app.controller.sub.post.create);
}
```

### 2. 自定义Controller 基类
我们可以通过自定义基类, 来实现一些Controller 公用的属性和方法
自定义基类的存储的地址为: app/core/**
```
// app/core/base_controller.js
const { Controller } = require('egg');
class BaseController extends Controller {
  get user() {
    return this.ctx.session.user;
  }

  success(data) {
    this.ctx.body = {
      success: true,
      data,
    };
  }

  notFound(msg) {
    msg = msg || 'not found';
    this.ctx.throw(404, msg);
  }
}
module.exports = BaseController;

//app/controller/post.js
const Controller = require('../core/base_controller');
class PostController extends Controller {
  async list() {
    const posts = await this.service.listByUser(this.user);
    this.success(posts);
  }
}
```

## 2.3 egg.Controller 提供的对象
egg.Controller 提供的对象会挂载在类的this 属性上
1. this.ctx: 当前请求的上下文 Context 对象的实例，通过它我们可以拿到框架封装好的处理当前请求的各种便捷属性和方法。
2. this.app: 当前应用 Application 对象的实例，通过它我们可以拿到框架提供的全局对象和方法。
3. this.service：应用定义的 Service，通过它我们可以访问到抽象出的业务层，等价于 this.ctx.service 。
4. this.config：应用运行时的配置项。
5. this.logger：logger 对象，上面有四个方法（debug，info，warn，error），分别代表打印四个不同级别的日志，使用方法和效果与 context logger 中介绍的一样，但是通过这个 logger 对象记录的日志，在日志前面会加上打印该日志的文件路径，以便快速定位日志打印位置。

# 3. HTTP 基础
## 3.1 介绍
Controller 基本上是业务开发中唯一和 HTTP 协议打交道的地方

HTTP 协议中并不建议在通过 GET、HEAD 方法访问时传递 body

框架内置了 bodyParser 中间件来对这两类格式的请求 body 解析成 object 挂载到 ctx.request.body 上

## 3.2 提取参数
1. this.ctx.query: 用于url 中非重复参数的提取
2. this.ctx.queries: 用于url 中重复参数的提取
3. this.ctx.params: 用于动态路由参数的提取
4. this.ctx.request.body: 用于请求中body 参数的提取
5. this.ctx.body: 用于响应中body 内容的设置, 和this.ctx.response.body 一致
6. this.ctx.response.body: 用于响应中body 内容的设置, 和this.ctx.body 一致

## 3.3 egg 框架的bodyParser 
### 1. 特性
bodyParser 在egg 框架中具有如下的特性
1. 当请求的 Content-Type 为 application/json，application/json-patch+json，application/vnd.api+json 和 application/csp-report 时，会按照 json 格式对请求 body 进行解析，并限制 body 最大长度为 100kb。
2. 当请求的 Content-Type 为 application/x-www-form-urlencoded 时，会按照 form 格式对请求 body 进行解析，并限制 body 最大长度为 100kb。
3. 如果解析成功，body 一定会是一个 Object（可能是一个数组）。

### 2. 配置
如果我们需要对bodyParser 进行配置, 那么我们可以在config/config.default.js 中覆盖系统的默认值
```
module.exports = {
  bodyParser: {
    jsonLimit: '1mb',
    formLimit: '1mb',
  },
};
```

### 3. 注意点
1. 如果用户的请求 body 超过了我们配置的解析最大长度，会抛出一个状态码为 413 的异常，如果用户请求的 body 解析失败（错误的 JSON），会抛出一个状态码为 400 的异常
2. 在调整 bodyParser 支持的 body 长度时，如果我们应用前面还有一层反向代理（Nginx），可能也需要调整它的配置，确保反向代理也支持同样长度的请求 body

## 3.4 header
1. this.ctx.headers / this.ctx.header / this.ctx.request.headers / this.ctx.request.header: 这几个属性等价, 用于获取整个header 对象
2. this.ctx.get(name) / this.ctx.request.get(name) : 获取请求 header 中的一个字段的值，如果这个字段不存在，会返回空字符串
3. this.ctx.host: 优先读通过 config.hostHeaders 中配置的 header 的值，读不到时再尝试获取 host 这个 header 的值，如果都获取不到，返回空字符串, config.hostHeaders 默认配置为 x-forwarded-host
4. this.ctx.protocol: 通过这个 Getter 获取 protocol 时，首先会判断当前连接是否是加密连接，如果是加密连接，返回 https。如果处于非加密连接时，优先读通过 config.protocolHeaders 中配置的 header 的值来判断是 HTTP 还是 https，如果读取不到，我们可以在配置中通过 config.protocol 来设置兜底值，默认为 HTTP。config.protocolHeaders 默认配置为 x-forwarded-proto
5. this.ctx.ips: 通过 ctx.ips 获取请求经过所有的中间设备 IP 地址列表，只有在 config.proxy = true 时，才会通过读取 config.ipHeaders 中配置的 header 的值来获取，获取不到时为空数组, config.ipHeaders 默认配置为 x-forwarded-for
6. this.ctx.ip: 通过 ctx.ip 获取请求发起方的 IP 地址，优先从 ctx.ips 中获取，ctx.ips 为空时使用连接上发起方的 IP 地址。 
7. this.ctx.status: 设置响应的状态码
8. this.ctx.body: 表示响应的body 内容, 和this.ctx.response.body 一致
9. this.ctx.response.body: 表示响应的body 内容, 和this.ctx.body 一致
10. this.ctx.request.body: 表示请求的body 内容, 一般存储着前端传递过来的参数

PS:
1. 推荐使用this.ctx.get(name) 获取header 的属性值, 而不是this.ctx.headers['name'], 因为前者会自动处理大小写
2. 

## 3.5 cookie
### 1. 定义
由于http 是无状态协议; 因此设置了一个cookie 头部, 
cookie在服务端设置的响应头部, 用于标记用户的身份, 之后的用户请求会自动带上该头部, 这样浏览器就知道访问者到底是谁

### 2. demo
```
class CookieController extends Controller {
  async add() {
    const ctx = this.ctx;
    let count = ctx.cookies.get('count');
    count = count ? Number(count) : 0;
    ctx.cookies.set('count', ++count);
    ctx.body = count;
  }

  async remove() {
    const ctx = this.ctx;
    const count = ctx.cookies.set('count', null);
    ctx.status = 204;
  }
}
```

### 3. 配置
```
module.exports = {
  cookies: {
    // httpOnly: true | false,
    // sameSite: 'none|lax|strict',
  },
};
```

### 4. 注意点
1. Cookie 虽然在 HTTP 中只是一个头，但是通过 foo=bar;foo1=bar1; 的格式可以设置多个键值对。
2. Cookie 在 Web 应用中经常承担了传递客户端身份信息的作用，因此有许多安全相关的配置
3. [cookie 文档](https://eggjs.org/zh-cn/core/cookie-and-session.html#cookie)

## 3.6 session
### 1. 定义
通过 Cookie，我们可以给每一个用户设置一个 Session，用来存储用户身份相关的信息，这份信息会加密后存储在 Cookie 中，实现跨请求的用户身份保持。

egg 框架内置了 Session 插件，给我们提供了 ctx.session 来访问或者修改当前用户 Session 

### 2. demo
```
class PostController extends Controller {
  async fetchPosts() {
    const ctx = this.ctx;
    // 获取 Session 上的内容
    const userId = ctx.session.userId;
    const posts = await ctx.service.post.fetch(userId);
    // 修改 Session 的值
    ctx.session.visited = ctx.session.visited ? ++ctx.session.visited : 1;
    ctx.body = {
      success: true,
      posts,
    };
  }
}
```

### 3. 配置
```
module.exports = {
  key: 'EGG_SESS', // 承载 Session 的 Cookie 键值对名字
  maxAge: 86400000, // Session 的最大有效时间
};
```

# 4. 文件上传
## 4.1 egg 中文件传输
对于文件来说, 一般通过Multipart/form-data 格式发送

egg 框架通过内置 Multipart 插件来支持获取用户上传的文件

## 4.2 file 模式
### 1. 启用file 模式
```
// config/config.default.js
exports.multipart = {
  mode: 'file',
};
```

### 2. 单个文件
前端代码:
```
<form method="POST" action="/upload?_csrf={{ ctx.csrf | safe }}" enctype="multipart/form-data">
  title: <input name="title" />
  file: <input name="file" type="file" />
  <button type="submit">Upload</button>
</form>
```

后端代码:
```
// app/controller/upload.js
const Controller = require('egg').Controller;
// 注意在egg 框架中文件模块的引入, 和Node.js 中不太一样
const fs = require('mz/fs');

module.exports = class extends Controller {
  async upload() {
    const { ctx } = this;
    // 获取文件的内容
    const file = ctx.request.files[0];
    const name = 'egg-multipart-test/' + path.basename(file.filename);
    let result;
    // 文件的处理流程
    try {
      // 处理文件，比如上传到云端
      result = await ctx.oss.put(name, file.filepath);
    } finally {
      // 需要删除临时文件
      await fs.unlink(file.filepath);
    }

    // 返回响应
    ctx.body = {
      url: result.url,
      // 获取所有的字段值
      requestBody: ctx.request.body,
    };
  }
};
```

### 3. 多个文件
多文件处理的核心内容就是遍历处理所有的文件内容
前端代码:
```
<form method="POST" action="/upload?_csrf={{ ctx.csrf | safe }}" enctype="multipart/form-data">
  title: <input name="title" />
  file1: <input name="file1" type="file" />
  file2: <input name="file2" type="file" />
  <button type="submit">Upload</button>
</form>
```

后端代码:
```
// app/controller/upload.js
const Controller = require('egg').Controller;
const fs = require('mz/fs');

module.exports = class extends Controller {
  async upload() {
    const { ctx } = this;
    console.log(ctx.request.body);
    console.log('got %d files', ctx.request.files.length);
    // 如果上传多个文件, 那么ctx.request.files 将会是一个可遍历的数据
    for (const file of ctx.request.files) {
      console.log('field: ' + file.fieldname);
      console.log('filename: ' + file.filename);
      console.log('encoding: ' + file.encoding);
      console.log('mime: ' + file.mime);
      console.log('tmp filepath: ' + file.filepath);
      let result;
      try {
        // 处理文件，比如上传到云端
        result = await ctx.oss.put('egg-multipart-test/' + file.filename, file.filepath);
      } finally {
        // 需要删除临时文件
        await fs.unlink(file.filepath);
      }
      console.log(result);
    }
  }
};
```

## 4.3 stream 模式
### 1. 介绍
stream 模式其实之前接触过:
1. gulp 插件就是使用stream 流的形式处理文件的(stream, though, though2)
2. Node.js 爬虫下载内容也是流文件的形式

### 2. 单个文件
前端代码
```
<form method="POST" action="/upload?_csrf={{ ctx.csrf | safe }}" enctype="multipart/form-data">
  title: <input name="title" />
  file: <input name="file" type="file" />
  <button type="submit">Upload</button>
</form>
```

后端代码
```
const path = require('path');
// 注意引入stream-wormhole 库, 用于将文件流消费掉, 防止浏览器卡死
const sendToWormhole = require('stream-wormhole');
const Controller = require('egg').Controller;

class UploaderController extends Controller {
  async upload() {
    const ctx = this.ctx;
    // 使用this.ctx.getFileStream() 获取文件
    const stream = await ctx.getFileStream();
    const name = 'egg-multipart-test/' + path.basename(stream.filename);
    // 文件处理，上传到云存储等等
    let result;
    try {
      result = await ctx.oss.put(name, stream);
    } catch (err) {
      // 特别注意这个点
      // 必须将上传的文件流消费掉，要不然浏览器响应会卡死
      await sendToWormhole(stream);
      throw err;
    }

    ctx.body = {
      url: result.url,
      // 所有表单字段都能通过 `stream.fields` 获取到
      fields: stream.fields,
    };
  }
}

module.exports = UploaderController;
```

要通过 ctx.getFileStream 便捷的获取到用户上传的文件，需要满足两个条件
1. 只支持上传一个文件
2. 上传文件必须在所有其他的 fields 后面，否则在拿到文件流时可能还获取不到 fields

### 3. 多个文件
注意这种处理形式
后端文件:
```
const sendToWormhole = require('stream-wormhole');
const Controller = require('egg').Controller;

class UploaderController extends Controller {
  async upload() {
    const ctx = this.ctx;
    // 获取所有的流文件形式
    const parts = ctx.multipart();
    let part;
    // parts() 返回 promise 对象
    while ((part = await parts()) != null) {
      if (part.length) {
        // 这是 busboy 的字段
        console.log('field: ' + part[0]);
        console.log('value: ' + part[1]);
        console.log('valueTruncated: ' + part[2]);
        console.log('fieldnameTruncated: ' + part[3]);
      } else {
        if (!part.filename) {
          // 这时是用户没有选择文件就点击了上传(part 是 file stream，但是 part.filename 为空)
          // 需要做出处理，例如给出错误提示消息
          return;
        }
        // part 是上传的文件流
        console.log('field: ' + part.fieldname);
        console.log('filename: ' + part.filename);
        console.log('encoding: ' + part.encoding);
        console.log('mime: ' + part.mime);
        // 文件处理，上传到云存储等等
        let result;
        try {
          result = await ctx.oss.put('egg-multipart-test/' + part.filename, part);
        } catch (err) {
          // 必须将上传的文件流消费掉，要不然浏览器响应会卡死
          await sendToWormhole(part);
          throw err;
        }
        console.log(result);
      }
    }
    console.log('and we are done parsing the form!');
  }
}

module.exports = UploaderController;
```
 
 
## 4.4 框架默认的白名单
```
// images
'.jpg', '.jpeg', // image/jpeg
'.png', // image/png, image/x-png
'.gif', // image/gif
'.bmp', // image/bmp
'.wbmp', // image/vnd.wap.wbmp
'.webp',
'.tif',
'.psd',
// text
'.svg',
'.js', '.jsx',
'.json',
'.css', '.less',
'.html', '.htm',
'.xml',
// tar
'.zip',
'.gz', '.tgz', '.gzip',
// video
'.mp3',
'.mp4',
'.avi',
```

## 4.5 新增支持的扩展名
```
// config/config.default.js
module.exports = {
  multipart: {
    fileExtensions: [ '.apk' ] // 增加对 apk 扩展名的文件支持
  },
};
```

## 4.6 覆盖整个白名单
当重写了 whitelist 时，fileExtensions 不生效
```
// config/config.default.js
module.exports = {
  multipart: {
    whitelist: [ '.png' ], // 覆盖整个白名单，只允许上传 '.png' 格式
  },
};
```
 

# 5. 参数校验
## 5.1 定义
服务端接收到用户传递过来的数据时, 可能需要对数据进行验证

egg 框架中 Validate 插件提供便捷的参数校验机制

## 5.2 Validate 插件
插件的引入
```
// config/plugin.js
exports.validate = {
  enable: true,
  package: 'egg-validate',
};
```

参数验证
```
class PostController extends Controller {
  async create() {
    // 校验参数
    // 如果不传第二个参数会自动校验 `ctx.request.body`
    // 当校验异常时，会直接抛出一个异常，异常的状态码为 422，errors 字段包含了详细的验证不通过信息
    this.ctx.validate({
      title: { type: 'string' },
      content: { type: 'string' },
    });
  }
}
```

自行定义处理异常
```
class PostController extends Controller {
  async create() {
    const ctx = this.ctx;
    try {
      ctx.validate(createRule);
    } catch (err) {
      ctx.logger.warn(err.errors);
      ctx.body = { success: false };
      return;
    }
  }
};
```

## 5.3 自定义校验规则
### 1. 自定义
我们可以通过app.validator.addRule(type, check) 的方式新增自定义规则
```
// app.js
app.validator.addRule('json', (rule, value) => {
  try {
    // 如果符合规则, 那么则不返回任何值
    JSON.parse(value);
  } catch (err) {
    // 如果不符合规则, 则抛出一个异常
    return 'must be json string';
  }
});
```

### 2. 使用自定义规则
```
class PostController extends Controller {
  async handler() {
    const ctx = this.ctx;
    // query.test 字段必须是 json 字符串
    const rule = { test: 'json' };
    ctx.validate(rule, ctx.query);
  }
};
```

# 6. 发送HTTP 响应
## 6.1 设置status
框架提供了一个简便的setter 来进行状态码的设置: 通过this.ctx.status 进行状态码的设置
```
class PostController extends Controller {
  async create() {
    // 设置状态码为 201
    this.ctx.status = 201;
  }
};
```

## 6.2 Content-Type 的设置
1. 作为一个 RESTful 的 API 接口 controller，我们通常会返回 Content-Type 为 application/json 格式的 body，内容是一个 JSON 字符串。
2. 作为一个 html 页面的 controller，我们通常会返回 Content-Type 为 text/html 格式的 body，内容是 html 代码段。

## 6.3 设置body
### 1. 一般响应数据
```
class ViewController extends Controller {
  async show() {
    this.ctx.body = {
      name: 'egg',
      category: 'framework',
      language: 'Node.js',
    };
  }
}
```

### 2. 响应html 数据
```
class ViewController extends Controller {
  async page() {
      this.ctx.body = '<html><h1>Hello</h1></html>';
   }
}
```

### 3. 响应流文件
由于 Node.js 的流式特性，我们还有很多场景需要通过 Stream 返回响应
```
class ProxyController extends Controller {
  async proxy() {
    const ctx = this.ctx;
    const result = await ctx.curl(url, {
      // 这里直接将接受的内容转换成流的形式
      streaming: true,
    });
    ctx.set(result.header);
    // result.res 是一个 stream
    ctx.body = result.res;
  }
};
```

### 4. 渲染模板
在egg 框架中, 我们通常不会手写html 页面, 而是会通过模板引擎生成
```
class HomeController extends Controller {
  async index() {
    const ctx = this.ctx;
    // 使用this.ctx.render() 来渲染模板引擎
    await ctx.render('home.tpl', { name: 'egg' });
    // ctx.body = await ctx.renderString('hi, {{ name }}', { name: 'egg' });
  }
};
```

## 6.4 设置header
通过this.ctx.set(key, value) 来设置响应的单个header
通过this.ctx.set(headers) 来设置响应的多个header
```
// app/controller/api.js
class ProxyController extends Controller {
  async show() {
    const ctx = this.ctx;
    const start = Date.now();
    ctx.body = await ctx.service.post.get();
    const used = Date.now() - start;
    // 设置一个响应头
    ctx.set('show-response-time', used.toString());
  }
};
```

## 6.5 重定向
### 1. 定义
框架通过 security 插件覆盖了 koa 原生的 ctx.redirect 实现重定向

### 2. API
1. this.ctx.redirect(url) 如果不在配置的白名单域名内，则禁止跳转。
2. this.ctx.unsafeRedirect(url) 不判断域名，直接跳转，一般不建议使用，明确了解可能带来的风险后使用。

### 3. 重定向的配置
若用户没有配置 domainWhiteList 或者 domainWhiteList数组内为空，则默认会对所有跳转请求放行，即等同于ctx.unsafeRedirect(url)
```
// config/config.default.js
exports.security = {
  domainWhiteList:['.domain.com'],  // 安全白名单，以 . 开头
};
```

# 7. jsonp
## 7.1 场景描述
有时, 我们需要给非本域的页面提供接口服务, 但是由于历史原因无法通过cors 实现, 所以此时采用jsonp 的方式解决跨域问题

## 7.2 路由声明
```
// app/router.js
module.exports = app => {
  const jsonp = app.jsonp();
  app.router.get('/api/posts/:id', jsonp, app.controller.posts.show);
  app.router.get('/api/posts', jsonp, app.controller.posts.list);
};
```

## 7.3 controller 声明
在 Controller 中，只需要正常编写即可
```
// app/controller/posts.js
class PostController extends Controller {
  async show() {
    this.ctx.body = {
      name: 'egg',
      category: 'framework',
      language: 'Node.js',
    };
  }
}
```

用户请求对应的 URL 访问到这个 controller 的时候，如果 query 中有 _callback=fn 参数，将会返回 JSONP 格式的数据，否则返回 JSON 格式的数据。
PS: 此时返回JSON 数据有意义吗? 前端页面又拿不到这些数据

## 7.4 jsonp 全局配置
```
// config/config.default.js
exports.jsonp = {
  callback: 'callback', // 识别 query 中的 `callback` 参数
  limit: 100, // 函数名最长为 100 个字符
};
```

## 7.5 jsonp 路由配置
```
// app/router.js
module.exports = app => {
  const { router, controller, jsonp } = app;
  router.get('/api/posts/:id', jsonp({ callback: 'callback' }), controller.posts.show);
  router.get('/api/posts', jsonp({ callback: 'cb' }), controller.posts.list);
};
```

## 7.6 跨站防御配置
### 1. 打开CSRF 配置
```
// config/config.default.js
module.exports = {
  jsonp: {
    csrf: true,
  },
};
```

### 2. CSRF 注意点
1. CSRF 校验依赖于 security 插件提供的基于 Cookie 的 CSRF 校验
2. CSRF 主要用于同域检测, 客户端在发起 JSONP 请求时，也要带上 CSRF token

### 3. referrer 配置
referrer 配置规则
1. 正则表达式
2. 字符串: .开头的为该域名和其子域名; 不以. 开头则表示当前域名
3. 数组
```
//config/config.default.js
exports.jsonp = {
  whiteList: /^https?:\/\/test.com\//,
  // whiteList: '.test.com',
  // whiteList: 'sub.test.com',
  // whiteList: [ 'sub.test.com', 'sub2.test.com' ],
};
```

### 4. 注意点
1. 当 CSRF 和 referrer 校验同时开启时，请求发起方只需要满足任意一个条件即可通过 JSONP 的安全校验






