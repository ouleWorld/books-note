[toc]

服务(service)

# 1. 定义
Service 就是在复杂业务场景下用于做业务逻辑封装的一个抽象层，提供这个抽象有以下几个好处：

1. 保持 Controller 中的逻辑更加简洁, controller 只需要关注流程即可, 不需要关心细节
2. 保持业务逻辑的独立性，抽象出来的 Service 可以被多个 Controller 重复调用。
3. 将逻辑和展现分离，更容易编写测试用例

可以把service 看作是数据层 + 数据处理层

# 2. 使用场景
1. 复杂的数据处理, 比如要展现的信息需要从数据库获取，还要经过一定的规则计算，才能返回用户显示。或者计算完成后，更新到数据库。
2. 第三方服务的调用, 比如要向另外一台服务器请求数据

# 3. 属性
1. this.ctx
2. this.app
3. this.service
4. this.config
5. this.logger

# 4. this.ctx 属性
1. this.ctx.curl 发起网络调用。
2. this.ctx.service.otherService 调用其他 Service。
3. this.ctx.db 发起数据库调用等， db 可能是其他插件提前挂载到 app 上的模块。

# 5. 注意点
1. Service 文件必须放在 app/service 目录，可以支持多级目录，访问的时候可以通过目录名级联访问。
2. 一个 Service 文件只能包含一个类， 这个类需要通过 module.exports 的方式返回。
3. Service 需要通过 Class 的方式定义，父类必须是 egg.Service。
4. Service 不是单例，是 请求级别 的对象 ，Service 是懒加载的，只有当访问到它的时候框架才会去实例化它, 框架在每次请求中首次访问 ctx.service.xx 时延迟实例化，所以 Service 中可以通过 this.ctx 获取到当前请求的上下文。

# 6. demo
```
// app/router.js
module.exports = app => {
  app.router.get('/user/:id', app.controller.user.info);
};

// app/controller/user.js
const Controller = require('egg').Controller;
class UserController extends Controller {
  async info() {
    const { ctx } = this;
    const userId = ctx.params.id;
    const userInfo = await ctx.service.user.find(userId);
    ctx.body = userInfo;
  }
}
module.exports = UserController;

// app/service/user.js
const Service = require('egg').Service;
class UserService extends Service {
  // 默认不需要提供构造函数。
  // constructor(ctx) {
  //   super(ctx); 如果需要在构造函数做一些处理，一定要有这句话，才能保证后面 `this.ctx`的使用。
  //   // 就可以直接通过 this.ctx 获取 ctx 了
  //   // 还可以直接通过 this.app 获取 app 了
  // }
  async find(uid) {
    // 假如 我们拿到用户 id 从数据库获取用户详细信息
    const user = await this.ctx.db.query('select * from user where uid = ?', uid);

    // 假定这里还有一些复杂的计算，然后返回需要的信息。
    const picture = await this.getPicture(uid);

    return {
      name: user.user_name,
      age: user.age,
      picture,
    };
  }

  async getPicture(uid) {
    const result = await this.ctx.curl(`http://photoserver/uid=${uid}`, { dataType: 'json' });
    return result.data;
  }
}
module.exports = UserService;

// curl http://127.0.0.1:7001/user/1234
```