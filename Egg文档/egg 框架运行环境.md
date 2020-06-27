[toc]

egg 框架运行环境

# 1. 什么是运行环境
在一个web 应用中, web 应用本身是无状态的, 其应该拥有根据运行环境设置自身的能力

# 2. 怎么指定运行环境
## 2.1 通过config/env 指定
可以通过config 目录下的文件内容来指定运行环境, 如 prod。一般通过构建工具来生成这个文件
```
// config/env
prod
```

## 2.2 通过EGG_SERVER_ENV 指定
比如, 当在生产环境启动应用时:
```
EGG_SERVER_ENV=prod npm start
```

# 3. 获取运行环境
egg 框架中, 可以通过app.config.env 来获取当前的运行环境

# 4. 运行环境变量
## 4.1 Nde.js egg.js 的环境变量
在Node.js 中, 一般会使用NODE_ENV 来区分运行环境
在egg.js 中, 一般会使用EGG_SERVER_ENV 来区分运行环境

## 4.2 运行环境的分类
分类: 本地开发环境, 测试环境, 生产环境

除了本地开发环境和测试环境外，其他环境可统称为服务器环境

## 4.3 NODE_ENV 和 EGG_SERVER_ENV 映射关系
| NODE_ENV | EGG_SERVER_ENV | 说明 |
| :----:| :----: | :----: |
|  | local | 本地开发环境 |
| test | unittest | 单元测试 |
| production | prod | 生产环境 |

## 4.4 与koa 的区别
在koa 中, 通过app.env 来获取环境变量
在egg 中, 通过app.config.env 来获取环境变量

# 5. config 配置
## 5.1 config 配置的解决方案
使用代码管理配置，在代码中添加多个环境的配置，在启动时传入当前环境的参数即可。但无法全局配置，必须修改代码

配置的变更也应该经过 review 后才能发布

## 5.2 多环境配置
当指定 env 时会同时加载对应的配置文件，并覆盖默认配置文件的同名配置。如 prod 环境会加载 config.prod.js 和 config.default.js 文件，config.prod.js 会覆盖 config.default.js 的同名配置

1. config.default.js: 默认的配置文件，所有环境都会加载这个配置文件，一般也会作为开发环境的默认配置文件 
2. config.prod.js: 生产环境配置
3. config.unittest.js: 测试环境配置
4. config.local.js: 本地测试环境配置

```
config
|- config.default.js 
|- config.prod.js
|- config.unittest.js
`- config.local.js
```

## 5.3 配置的写法
### 1. module.export 写法
```
// 配置 logger 文件的目录，logger 默认配置由框架提供
module.exports = {
  logger: {
    dir: '/home/admin/logs/demoapp',
  },
};
```

### 2. export.key 写法
```
exports.keys = 'my-cookie-secret-key';
exports.logger = {
  level: 'DEBUG',
};
```

### 3. function 写法
appInfo.root 是一个优雅的适配，比如在服务器环境我们会使用 /home/admin/logs 作为日志目录，而本地开发时又不想污染用户目录，这样的适配就很好解决这个问题。

appInfo 参数值
| appInfo | 说明 |
| :----:| :----: |
| pkg | package.json |
| name | 应用名，同 pkg.name |
| baseDir | 应用代码的目录 |
| HOME | 用户目录，如 admin 账户为 /home/admin |
| root | 应用根目录，只有在 local 和 unittest 环境下为 baseDir，其他都为 HOME |

```
// 将 logger 目录放到代码目录下
const path = require('path');
module.exports = appInfo => {
  return {
    logger: {
      dir: path.join(appInfo.baseDir, 'logs'),
    },
  };
};
```

### 4. 非常错误的写法
```
// config/config.default.js
exports.someKeys = 'abc';
module.exports = appInfo => {
  const config = {};
  config.keys = '123456';
  return config;
};
```

## 5.4 配置加载顺序
应用、插件、框架都可以定义这些配置

在生产环境下, 它们加载的顺序为
```
-> 插件 config.default.js
-> 框架 config.default.js
-> 应用 config.default.js
-> 插件 config.prod.js
-> 框架 config.prod.js
-> 应用 config.prod.js
```

## 5.5 合并规则
配置之间的合并采用的规则是直接覆盖, 而非进行合并
```
const a = {
  arr: [ 1, 2 ],
};
const b = {
  arr: [ 3 ],
};
extend(true, a, b);
// => { arr: [ 3 ] }
```

## 5.6 配置的结果
框架在启动时会把合并后的最终配置 dump 到 run/application_config.json（worker 进程）和 run/agent_config.json（agent 进程）中，可以用来分析问题

还会生成 run/application_config_meta.json（worker 进程）和 run/agent_config_meta.json（agent 进程）文件，用来排查属性的来源

配置文件中会隐藏一些字段
1. 如密码、密钥等安全字段，这里可以通过 config.dump.ignore 配置，必须是 Set 类型
2. 如函数、Buffer 等类型，JSON.stringify 后的内容特别大



