##### 框架的特色：以文件目录路径为解析单元

##### 一.  最基础的服务：router + Controller

**`router`**  用来描述请求URL与具体承当执行动作的Controller的对应关系

> router.requestMethos(path, Controller)

```js
//e.g. info为controller/user.js中的info方法
router.get('user/:id', controller.user.info)
```

**`Controller`**  解析用户输入，处理后返回相应的结果。

要做的事情：获取HTTP请求参数，制定参数规则，校验参数，组装参数，业务处理，设置响应内容和响应状态。

属性：ctx, app, config, service, logger



Controller要做的事情太多了，为了简洁性，将业务处理这步派发出去，于是有了**`Service`**



##### 二. 铁三角： router + Controller + Service 成立了

**`Service`** 专门处理业务逻辑，保持Controller的简洁，也便于管理

1. 网络的调用  this.ctx.curl   (相当于ajax请求)
2. 其他service的调用   this.ctx.service.otherService   （调用其他服务的结果）
3. 数据库的调用   this.ctx.db   （后台语言数据库操作肯定也少不了）

属性上和Controller一样，



业务往往是比较复杂的，而且还会有一些重复的函数方法之类的，为了处理这一部分，有了**`Middleware`**



##### 三.  Middleware中间件(不是中间层)

**`Middleware`**  就是工具函数，帮助更好的处理数据。基于洋葱模型，每使用一个中间件就相当于在外面套了一层，所以，它是有顺序关系的。



有顺序关系就会导致不同的顺序不同的结果，出错就难免会发生，然后又多了一个东西：**插件**



##### 四.   插件，开箱即用，迷你的一个应用，中间件是自己造车，插件就是一辆完整的车。

除了中间件和插件(这两个相当于一个)外，还有**`extend`**

##### 五.   extend 相当于过滤函数，原型的一些方法，属性的setter,getter，也是帮助处理数据,与中间件不同的是，中间件有点类似处理一个整体流程，把握大方向，extend有点处理局部，在细节处雕琢。



目前有了：router + controller + service + middleware + 插件 + extend常规业务需求已经覆盖到了

另外总有些特殊需求，比如**定时任务**。



##### 六.  定时任务，帮助我们处理一些持续性，周期性的工作，上报状态，清理缓存等

##### 七.配置Config，提倡配置和代码分离，便于统一管理和清晰脉络

##### 八.   应用生命周期函数（命名风格类似React），app.js

**`configWillLoad`**  : 此时config文件已经被读取并合并，还未生效，应用层最后修改配置机会。只支持同步调用

**`configDidLoad`** ：配置文件加载完成

**`didLoad`** ：文件加载完成，可以用来加载应用自定义文件，启动自定义服务

**`willReady`** ：插件启动完毕，应用整体还未ready，可以做一些数据初始化操作

**`didReady`** ：woker准备就绪，应用启动完毕

**`serverDidReady`** ：http server已启动，开始接受玩不请求，可以从app.server拿到server实例

**`beforeClose`** ：应用即将关闭

##### 九. 单元测试

1. 保障代码质量
2. 正确性保障
3. 增强自信心
4. 应用的Controller,Service,Helper,Extend等代码尽量要求100%覆盖到

##### 其他

1. 日志
2. 错误上报
3. 安全
4. 部署



 



