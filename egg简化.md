

#### 基础

~~~js
application  ----   在Service和Controller里可以通过this.app调用
					this.ctx.app可以访问app,
                    在Application实例，第一个参数就是app
					
context      ----   在Middleware中，第一个参数就是ctx。
					在Controller和Service里，通过this.ctx调用

request      ----	this.ctx.request
    				[get]  ctx.request.query === ctx.query
					[post] ctx.request.body
                    [FormData] ctx.request.file

response     ----	this.ctx.response
					ctx.response.body === ctx.body
					另外，ctx.status 设置响应状态
                    ctx.set()函数设置响应头

Controller   ----	基类里this上挂载着ctx,app,config,service,logger

Service      ----	与Controller基类属性一致

Helper       ----	工具函数，在extend文件里。编写helper.js
					ctx.helper，或者模板里直接使用helpe.xx

Config       ----	在Application实例，app.config
					在Controller和Service基类里this.config

logger       ----	日志级别: app.logger,ctx.logger
					在Controller和Service基类里this.logger
					日志类型: debug, info, warn, error都是函数

~~~

#### 中间件

* app.middleware  =  app.config.appMiddleware + app.config.coreMiddleware

* middleware文件夹下编写中间件，接收两个参数，options和app，options对象包含了，config文件里对应中间件的配置项，返回一个异步函数，函数接收ctx和next两个参数 

  ~~~js
  // app/config/config.default.js
  module.exports = {
      // 需要加载的中间件放数组里，全局性，每次都会调用
      // 针对路由单独生效则在路由文件里app.middleware单独调用
      // middleware函数里next位置一致情况下，根据数组顺序决定执行顺序
      middleware: ['auth'],
      
      auth: {
          // 每个中间件都有三个通用配置，单独配置不谈
          // enable，ignore, match, ignore
          // match和ignore二选一，顾名思义不解释
          // match和ingore支持字符串，正则，和函数
          enable: true, // 默认true开启
          ignore(ctx) { // 忽略请求走该中间件
              // TODO
          }
      }
  }
  
  // app/middleware/auth.js
  module.exports = (options,  app) => {
      return async function auth(ctx, next) {
          // 1. await next()位置放最前表示在承接，所以调用该中间件
          // 的时机在controller,service处理完正常逻辑之后
          
          // TODO
          // options.ignore 对应上面配置文件里的ingore函数
          
          // 2. await next()位置放最后表示启下，所以调用该中间件
          // 的时机在controller,service处理完正常逻辑之前
          
          // 举例，auth中间件一般是判断请求者有没有权限，反正逻辑之前
          // 所以await next()根据条件放最后，给过就next，不给
          // 就返回状态码401.
          if (!ctx.session.userId) {
              ctx.status = 401;
          } else {
              await next();
          }
      }
  }
  ~~~

#### 定时任务

* 位置 app/schedule 目录，每个文件都是单独定时任务

  ~~~js
  // 可以为对象或函数，为函数时，接收app为参数
  // module.exports = app => {
  //   return {
  //		同结构
  //   }
  // }
  module.exports = {
      schedule: {
        interval: '1m', // interval 或 cron
        type: 'all', // all|work,all：每台机器每个worker都执行，worker：每台机器随机一个worker执行
        // cron: '0 0 */3 * * *', // 每三小时准点执行一次
      },
      async task(ctx) {
          // TODO
      }
  }
  // interval 支持 String|Number, Number单位为毫秒
  // cron 设置执行时机稍微复杂点
  // * * * * * *   分别代表 秒，分，时，日期，月份，周
  // 其他参数：
  //    cronOptions： 参见cron-parser
  //    immediate: true时，应用启动并ready就立刻执行一次该定时任务
  //    disable：禁止该任务启动
  //    env: Array， 仅在指定环境下执行
  
  
  //   通过app.runSchedule(schedulePath)可以手动运行定时任务，返回promise
  ~~~

#### 扩展

* app/extend文件下，五个文件

* application.js / content.js / request.js / response.js / helper.js

  ~~~js
  module.exports = {
      xxx(params) {
          // this对应各个对象，看什么文件
      }
  }
  ~~~

  

* 相当与工具函数，挂载在不同的对象上，app, ctx, ctx.request, ctx.response，ctx.helper来调用工具函数

* 如果要根据环境来扩展，则修改文件名，如application.product.js就是在product环境下才有文件里的工具函数

#### 路由

* 格式： **router.verb(~~[routerName,]~~ pathName, ~~[middleware1, ...middlerware99,]~~ controller)**

  `verb:`  head, options, get, post, put, patch, del, redirect

  `routerName:`  可选，路由设定别名，没用过。

  `pathName:`  请求路径。支持正则。

  `middleware:` 针对路由可以单独设置中间件，可串联多个。

  `controller:`  可以指定具体controller，如app.controller.user.index,或者字符串形式，如'user.index';

  ~~~js
  // app/router.js
  module.exports = app => {
      const { router, controller } = app;
      const prefix = '/mkv/';
      router.get(`${prefix}user/logstate`, controller.user.login)
  }
  
  // app/controller/user.js
  class UserController extends Controller {
      async login() {
          // TODO
      }
  }
  ~~~

* 重定向

  ~~~js
  // 第一种，路由文件配置
  router.redirect('/', '/home/index', 302);
  // 第二种，controller或service逻辑里
  ctx.redirect('https://www.baidu.com');
  ~~~

* 路由文件分包

  ~~~js
  // app/router.js
  module.exports = app => {
      require('./router/news')(app);
  }
  // app/router/news.js
  module.exports = app => {
      app.router.get('new/detail', app.controller.new.detail);
  }
  ~~~

#### file上传

~~~js
// config.default.js
config.multipart = {
    fileSize: '100mb',
    mode: 'file',
    whitelist: ['.xlsx', '.csv', 'xls'],
}

// controller
ctx.request.files // 获取文件

// service， 对于xlsx文件处理,引入插件node-xlsx
const data = await fs.readFileSync(file.filepath)
const list = xlsx.parse(data)
const [head, ...tableData] = list[0].data; // 一行一个数组，第一行是head头

~~~

