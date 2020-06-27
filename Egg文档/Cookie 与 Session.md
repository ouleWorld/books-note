[toc]

Cookie 与 Session

# 1. cookie
## 1.1 什么是cookie
HTTP 请求都是无状态的，但是我们的 Web 应用通常都需要知道发起请求的人是谁。为了解决这个问题，HTTP 协议设计了一个特殊的请求头：Cookie。服务端可以通过响应头（set-cookie）将少量数据响应给客户端，浏览器会遵循协议将数据保存，并在下次请求同一个服务的时候带上（浏览器也会遵循协议，只在访问符合 Cookie 指定规则的网站时带上对应的 Cookie 来保证安全性）。

## 1.2 如果设置cookie
通过 ctx.cookies.set(key, value, options)
```
class HomeController extends Controller {
  async add() {
    const ctx = this.ctx;
    let count = ctx.cookies.get('count');
    count = count ? Number(count) : 0;
    ctx.cookies.set('count', ++count);
    ctx.body = count;
  }
  async remove() {
    const ctx = this.ctx;
    ctx.cookies.set('count', null);
    ctx.status = 204;
  }
}
```


## 1.2 如果获取cookie
### 1. 获取方式
通过 ctx.cookies.get(key, options)

### 2. 获取时需要注意点
服务端在设置cookie 时, 可能会设置 options.signed 和 options.encrypt, 我们在获取的时候需要遵循如下的规则
1. 如果设置的时候指定为 signed，获取时未指定，则不会在获取时对取到的值做验签，导致可能被客户端篡改
2. 如果设置的时候指定为 encrypt，获取时未指定，则无法获取到真实的值，而是加密过后的密文

### 3. 特殊情况
如果要获取前端或者其他系统设置的 cookie，需要指定参数 signed 为 false，避免对它做验签导致获取不到 cookie 的值
PS: 其他系统设置的cookie 为什么能被访问到? cookie不是对url 的么
```
ctx.cookies.get('frontend-cookie', {
  signed: false,
});
```


## 1.3 ctx.cookies.set(key, value, options) 配置
### 1. HTTP 协议配置
1. {Number} maxAge: 设置这个键值对在浏览器的最长保存时间。是一个从服务器当前时刻开始的毫秒数。
2. {Date} expires: 设置这个键值对的失效时间，如果设置了 maxAge，expires 将会被覆盖。如果 maxAge 和 expires 都没设置，Cookie 将会在浏览器的会话失效（一般是关闭浏览器时）的时候失效。
3. {String} path: 设置键值对生效的 URL 路径，默认设置在根路径上（/），也就是当前域名下的所有 URL 都可以访问这个 Cookie。
4. {String} domain: 设置键值对生效的域名，默认没有配置，可以配置成只在指定域名才能访问。
5. {Boolean} httpOnly: 设置键值对是否可以被 js 访问，默认为 true，不允许被 js 访问。
6. {Boolean} secure: 设置键值对只在 HTTPS 连接上传输，框架会帮我们判断当前是否在 HTTPS 连接上自动设置 secure 的值。


### 2. egg 框架新增配置
1. {Boolean} overwrite：设置 key 相同的键值对如何处理，如果设置为 true，则后设置的值会覆盖前面设置的，否则将会发送两个 set-cookie 响应头。
2. {Boolean} signed：设置是否对 Cookie 进行签名，如果设置为 true，则设置键值对的时候会同时对这个键值对的值进行签名，后面取的时候做校验，可以防止前端对这个值进行篡改。默认为 true。
3. {Boolean} encrypt：设置是否对 Cookie 进行加密，如果设置为 true，则在发送 Cookie 前会对这个键值对的值进行加密，客户端无法读取到 Cookie 的明文值。默认为 false。


## 1.4 注意点
1. 默认的配置下，Cookie 是加签不加密的，浏览器可以看到明文，js 不能访问，不能被客户端（手工）篡改
2. 由于浏览器和其他客户端实现的不确定性，为了保证 Cookie 可以写入成功，建议 value 通过 base64 编码或者其他形式 encode 之后再写入。
3. 由于浏览器对 Cookie 有长度限制限制，所以尽量不要设置太长的 Cookie。一般来说不要超过 4093 bytes。当设置的 Cookie value 大于这个值时，框架会打印一条警告日志。

## 1.5 cookie 密钥
### 1. 配置
```
// config/config.default.js
module.exports = {
  keys: 'key1,key2',
};
```

### 2. 加密解密的规则
1. 加密和加签时只会使用第一个秘钥。
2. 解密和验签时会遍历 keys 进行解密

举例: 如果我们想要更新 Cookie 的秘钥，但是又不希望之前设置到用户浏览器上的 Cookie 失效，可以将新的秘钥配置到 keys 最前面，等过一段时间之后再删去不需要的秘钥即可。

# 2. Session
## 2.1 定义
Cookie 在 Web 应用中经常承担标识请求方身份的功能，所以 Web 应用在 Cookie 的基础上封装了 Session 的概念，专门用做用户身份识别

## 2.2 Session 的访问
框架内置了 Session 插件，给我们提供了 ctx.session 来访问或者修改当前用户 Session

访问/修改Session:  
```
class HomeController extends Controller {
  async fetchPosts() {
    const ctx = this.ctx;
    // 获取 Session 上的内容
    const userId = ctx.session.userId;
    const posts = await ctx.service.post.fetch(userId);
    // 修改 Session 的值
    ctx.session.visited = ctx.session.visited ? (ctx.session.visited + 1) : 1;
    ctx.body = {
      success: true,
      posts,
    };
  }
}
```

删除Session
```
ctx.session = null;
```

## 2.3 Session 的配置
下面是Session 的默认配置: 
可以看到这些参数除了 key 都是 Cookie 的参数，key 代表了存储 Session 的 Cookie 键值对的 key 是什么
```
exports.session = {
  key: 'EGG_SESS',
  maxAge: 24 * 3600 * 1000, // 1 天
  httpOnly: true,
  encrypt: true,
};
```


## 2.4 Session + Cookie 存储方式的问题
前提: Session 对象过于庞大
1. 前面提到，浏览器通常都有限制最大的 Cookie 长度，当设置的 Session 过大时，浏览器可能拒绝保存。
2. Cookie 在每次请求时都会带上，当 Session 过大时，每次请求都要额外带上庞大的 Cookie 信息。


## 2.5 拓展存储
框架提供了将 Session 存储到除了 Cookie 之外的其他存储的扩展方案，我们只需要设置 app.sessionStore 即可将 Session 存储到指定的存储中。

例如 egg-session-redis 就提供了将 Session 存储到 redis 中的能力，在应用层，我们只需要引入 egg-redis 和 egg-session-redis 插件即可。

```
// app.js
module.exports = app => {
  app.sessionStore = {
    // support promise / async
    async get (key) {
      // return value;
    },
    async set (key, value, maxAge) {
      // set key to store
    },
    async destroy (key) {
      // destroy key
    },
  };
};
```


## 2.6 Session 实践 - 修改用户 Session 失效时间
虽然在 Session 的配置中有一项是 maxAge，但是它只能全局设置 Session 的有效期，我们经常可以在一些网站的登陆页上看到有 记住我 的选项框，勾选之后可以让登陆用户的 Session 有效期更长。这种针对特定用户的 Session 有效时间设置我们可以通过 ctx.session.maxAge= 来实现。
```
const ms = require('ms');
class UserController extends Controller {
  async login() {
    const ctx = this.ctx;
    const { username, password, rememberMe } = ctx.request.body;
    const user = await ctx.loginAndGetUser(username, password);

    // 设置 Session
    ctx.session.user = user;
    // 如果用户勾选了 `记住我`，设置 30 天的过期时间
    if (rememberMe) ctx.session.maxAge = ms('30d');
  }
}
```


## 2.7 Session 实践 - 延长用户 Session 有效期
默认情况下，当用户请求没有导致 Session 被修改时，框架都不会延长 Session 的有效期，但是在有些场景下，我们希望用户如果长时间都在访问我们的站点，则延长他们的 Session 有效期，不让用户退出登录态。框架提供了一个 renew 配置项用于实现此功能，它会在发现当用户 Session 的有效期仅剩下最大有效期一半的时候，重置 Session 的有效期。
```
// config/config.default.js
module.exports = {
  session: {
    renew: true,
  },
};
```


## 2.8 注意点
1. 不要以 _ 开头
2. 不能为 isNew
3. 在默认的配置下，存放 Session 的 Cookie 将会加密存储、不可被前端 js 访问，这样可以保证用户的 Session 是安全的。
4. 一旦选择了将 Session 存入到外部存储中，就意味着系统将强依赖于这个外部存储，当它挂了的时候，我们就完全无法使用 Session 相关的功能了。因此我们更推荐大家只将必要的信息存储在 Session 中，保持 Session 的精简并使用默认的 Cookie 存储，用户级别的缓存不要存储在 Session 中。
