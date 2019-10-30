

## WEB DEPLOY

> 基本参考 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin) ,另外 [D2-admin](https://github.com/d2-projects/d2-admin) 也非常nice。

#### 鉴权路由 Auth

 - router.beforeEach 每次判断权限

   ~~~js
   // 在路由meta信息里配置roles，作为判断依据
   // beforeEach里处理
   router.beforeEach(async (to, from ,next) => {
       // 拿到页面访问需要的身份
       const roles = to.meta.roles;
       // 判断用户是否已经登陆
       if (!store.state.user.logged) {
           // 获取用户身份
           const ticket = to.query.ticket
           const userInfo = await store.dispatch('user/role', ticket)
           const { code } = userInfo
           if (code === SUCCESS_CODE) {
               // 首页一般不需要权限，直接导入首页
               next('/')
           } else {
               // 弹窗，重新登陆处理
           }
       } else {
           if (roles && roles.length) {
               // 鉴权函数，判断用户身份里有没有meta里role定义的角色
               if (checkPermisson(roles)){
                   next();
               } else {
                   // 弹窗，重新登陆处理
               }
           } else {
              next() 
           }
       }
   })
   // checkPermisson.js
   export const checkPermisson = roles => {
       const userRoles = store.state.user.roles
       const hasAuth = roles.some(v => userRoles.includes(v));
       return hasAuth;
   }
   
   // permission.js,对于某些功能的权限处理，通过指令来完成
   // Vue.directive('permission', permission); 需要注册指令
   export default {
       inserted(el,binding) {
           const { value, arg } = binding
           // 数组，可能多身份
           const roles = store.state.user.roles
           if (value && value instanceof Array && value.lengh) {
               const hasAuth = value.some(v => roles.includes(v));
	            if (!hasAuth) {
	                el.parentNode && el.parentNode.removeChild(el)
	            }
	     } else {
	            throw new Error('need roles! Like v-permission="[\'admin\',\'editor\']"')
	     }
	    }
	}
	
	~~~
	
	
	
	- addRoutes && removeRoutes，添加权限路由方式
	
	  ~~~js
	  // 路由的注册，路由分公共路由和权限路由，公共路由直接加载，权限路由通过addRoutes函数去添加
	  const createRouter = () => new Router({
	    // mode: 'history', // require service support
	    scrollBehavior: () => ({ y: 0 }),
	    routes: constantRoutes
	  })
	  
	  const router = createRouter()
	  
	  // 重置路由
	  export function resetRouter() {
	    const newRouter = createRouter()
	    // matcher提供两个属性，match和addRouter，重新赋值使新路由生效
	    router.matcher = newRouter.matcher // reset router
	  }
	  // 添加路由
	  router.beforeEach(async(to, from, next) => {
	    // 获取cookie里的token
	    const hasToken = getToken()
	    // 判断是否有token
	    if (hasToken) {
	      if (to.path === '/login') {
	        // 有token去登录页就重定向到首页
	        next({ path: '/' })
	      } else {
	        // 获取用户角色
	        const hasRoles = store.getters.roles && store.getters.roles.length
	        if (hasRoles) {
	          next()
	        } else {
	          try {
	            // 没有角色的就要去请求用户信息
	            const { roles } = await store.dispatch('user/getInfo')
	  
	            // 根据用户信息里的role去拿可访问的路由
	            const accessRoutes = await store.dispatch('permission/generateRoutes', roles)
	  
	            // 添加路由菜单数组
	            router.addRoutes(accessRoutes)
	  
	            // 确保添加路由成功的hack方法
	            // 消除历史记录
	            next({ ...to, replace: true })
	          } catch (error) {
	            // 出错了就消除记录重新登陆
	            await store.dispatch('user/resetToken')
	            Message.error(error || 'Has Error')
	            next(`/login?redirect=${to.path}`)
	          }
	        }
	      }
	    } else {
	  	// 可以设置白名单允许通过
	      if (whiteList.indexOf(to.path) !== -1) {
	        next()
	      } else {
	        // 没有token又不在白名单就送去登陆
	        next(`/login?redirect=${to.path}`)
	      }
	    }
	  })
	  
	  // generateRoutes, vuex
	  const actions = {
	    generateRoutes({ commit }, roles) {
	      return new Promise(resolve => {
	        let accessedRoutes
	        if (roles.includes('admin')) {
	            // 最大权限者直接把所有权限路由添加
	          accessedRoutes = asyncRoutes || []
	        } else {
	            // 根据身份过滤路由
	          accessedRoutes = filterAsyncRoutes(asyncRoutes, roles)
	        }
	          // 将数据存储来更新菜单
	        commit('SET_ROUTES', accessedRoutes)
	        resolve(accessedRoutes)
	      })
	    }
	  }
	  // 路由数据还要用来处理菜单
	  const mutations = {
	    SET_ROUTES: (state, routes) => {
	      state.addRoutes = routes
	      state.routes = constantRoutes.concat(routes)
	    }
	  }
	  // 过滤工具函数
	  /**
	   * 判断是否有权限
	   * @param roles
	   * @param route
	   */
	  function hasPermission(roles, route) {
	    if (route.meta && route.meta.roles) {
	      return roles.some(role => route.meta.roles.includes(role))
	    } else {
	      return true
	    }
	  }
	  
	  /**
	   * 过滤权限路由
	   * @param routes asyncRoutes
	   * @param roles
	   */
	  export function filterAsyncRoutes(routes, roles) {
	    const res = []
	  
	    routes.forEach(route => {
	      const tmp = { ...route }
	      if (hasPermission(roles, tmp)) {
	        if (tmp.children) {
	          tmp.children = filterAsyncRoutes(tmp.children, roles)
	        }
	        res.push(tmp)
	      }
	    })
	  
	    return res
	  }
	  
	  ~~~
	  
	- 将权限路由存入服务端，根据token请求的时候，直接返回路由数组，此方法可配置角色权限能力，就是稍微麻烦一些，新增路由页面需要提交一份数据入库，接口请求也要在后端判断角色权限能力，做个中间件，其实感觉后端本身就要判断角色权限。

#### 图标 Svg, FontIcon

* font-class，直接使用iconfont创建一个项目，添加图标生成外链插入html就可以了，也可以下载到本地

* svg，推荐svg，支持多色，不失真。

  * 安装相关工具

    * ```
      yarn add svg-sprite-loader -D
      ```

    * ```
      npm install -g svgo   去掉svg标签里的多余信息
      ```

  * 文件目录，在src下创建目录形式如下

    ~~~
    icons
    ├─index.js
    ├─svgo.yml
    ├─svg
    |  ├─404.svg
    |  ├─bug.svg
    |  └zip.svg
    
    ~~~

    ```vue
    <template>
      <div v-if="isExternal" :style="styleExternalIcon" class="svg-external-icon svg-icon" v-on="$listeners" />
      <svg v-else :class="svgClass" aria-hidden="true" v-on="$listeners">
        <use :xlink:href="iconName" />
      </svg>
    </template>
    
    <script>
    // doc: https://panjiachen.github.io/vue-element-admin-site/feature/component/svg-icon.html#usage
    
    export default {
      name: 'SvgIcon',
      props: {
        iconClass: {
          type: String,
          required: true
        },
        className: {
          type: String,
          default: ''
        }
      },
      computed: {
        isExternal() {
          return /^(https?:|mailto:|tel:)/.test(this.iconClass)
        },
        iconName() {
          return `#icon-${this.iconClass}`
        },
        svgClass() {
          if (this.className) {
            return 'svg-icon ' + this.className
          } else {
            return 'svg-icon'
          }
        },
        styleExternalIcon() {
          return {
            mask: `url(${this.iconClass}) no-repeat 50% 50%`,
            '-webkit-mask': `url(${this.iconClass}) no-repeat 50% 50%`
          }
        }
      }
    }
    </script>
    
    <style scoped>
    .svg-icon {
      width: 1em;
      height: 1em;
      vertical-align: -0.15em;
      fill: currentColor;
      overflow: hidden;
    }
    
    .svg-external-icon {
      background-color: currentColor;
      mask-size: cover!important;
      display: inline-block;
    }
    </style>
    
    ```

    

    ~~~js
    // index.js
    import Vue from 'vue'
    import SvgIcon from '@/components/SvgIcon'// svg component
    
    // register globally
    Vue.component('svg-icon', SvgIcon)
    
    const req = require.context('./svg', false, /\.svg$/)
    const requireAll = requireContext => requireContext.keys().map(requireContext)
    // 返回数组里每个item都是module
    requireAll(req)
    ~~~

    ~~~yml
    # package.json 添加命令"svgo": "svgo -f src/icons/svg --config=src/icons/svgo.yml",
    # ----  svgo.yml  start-------
    # replace default config
    
    # multipass: true
    # full: true
    
    plugins:
    
      # - name
      #
      # or:
      # - name: false
      # - name: true
      #
      # or:
      # - name:
      #     param1: 1
      #     param2: 2
    
    - removeAttrs:
        attrs:
          - 'fill'
          - 'fill-rule'
    ~~~

    

  * vue-cli3 vue.config.js配置

    ~~~js
    // set svg-sprite-loader
    chainWebpack(config) {
      config.module
        .rule('svg')
        .exclude.add(resolve('src/icons'))
        .end()
      config.module
        .rule('icons')
        .test(/\.svg$/)
        .include.add(resolve('src/icons'))
        .end()
        .use('svg-sprite-loader')
        .loader('svg-sprite-loader')
        .options({
            symbolId: 'icon-[name]'
         })
         .end()
    }
    ~~~

    

#### 模拟数据 Mock Server

* 相关依赖

  * [Mock](https://github.com/nuysoft/Mock/wiki/Mock.mock()) 

    ~~~js
    // mock/index.js
    import Mock from 'mockjs'
    import { param2Obj } from '../src/utils'
    
    import user from './user'
    import role from './role'
    import article from './article'
    import search from './remote-search'
    
    const mocks = [
      ...user,
      ...role,
      ...article,
      ...search
    ]
    
    // for front mock
    // please use it cautiously, it will redefine XMLHttpRequest,
    // which will cause many of your third-party libraries to be invalidated(like progress event).
    export function mockXHR() {
      // mock patch
      // https://github.com/nuysoft/Mock/issues/300
      Mock.XHR.prototype.proxy_send = Mock.XHR.prototype.send
      Mock.XHR.prototype.send = function() {
        if (this.custom.xhr) {
          this.custom.xhr.withCredentials = this.withCredentials || false
    
          if (this.responseType) {
            this.custom.xhr.responseType = this.responseType
          }
        }
        this.proxy_send(...arguments)
      }
    
      function XHR2ExpressReqWrap(respond) {
        return function(options) {
          let result = null
          if (respond instanceof Function) {
            const { body, type, url } = options
            // https://expressjs.com/en/4x/api.html#req
            result = respond({
              method: type,
              body: JSON.parse(body),
              query: param2Obj(url)
            })
          } else {
            result = respond
          }
          return Mock.mock(result)
        }
      }
    
      for (const i of mocks) {
        Mock.mock(new RegExp(i.url), i.type || 'get', XHR2ExpressReqWrap(i.response))
      }
    }
    
    // for mock server
    const responseFake = (url, type, respond) => {
      return {
        url: new RegExp(`/mock${url}`),
        type: type || 'get',
        response(req, res) {
          res.json(Mock.mock(respond instanceof Function ? respond(req, res) : respond))
        }
      }
    }
    
    export default mocks.map(route => {
      return responseFake(route.url, route.type, route.response)
    })
    
    // main.js
    if (process.env.NODE_ENV === 'development') {
    import { mockXHR } from '../mock'
      mockXHR()
    }
    
    ~~~

    

    ~~~js
    // mock-server.js
    const chokidar = require('chokidar') // 监听文件改变，发生变化卸载接口路由重新注册
    const bodyParser = require('body-parser')
    const chalk = require('chalk') // 输出增色插件
    const path = require('path')
    
    // process.cwd() node命令执行时所在文件目录， __dirname是始终是被执行文件所在文件目录
    const mockDir = path.join(process.cwd(), 'mock')
    
    function registerRoutes(app) {
      let mockLastIndex
      const { default: mocks } = require('./index.js')
      for (const mock of mocks) { 
        app[mock.type](mock.url, mock.response)
        mockLastIndex = app._router.stack.length
      }
      const mockRoutesLength = Object.keys(mocks).length
      return {
        mockRoutesLength: mockRoutesLength,
        mockStartIndex: mockLastIndex - mockRoutesLength
      }
    }
    
    function unregisterRoutes() {
      // 在Node.js中，require.cache对象具有一个“键名/键值”结构，键名为每个模块的完整文件名，键值为各模块对象
      // 加载一个模块会，会缓存，再次调用直接返回缓存对象
      Object.keys(require.cache).forEach(i => {
        if (i.includes(mockDir)) {
          // 在Node.js中，可以使用require.resolve函数来查询某个模块文件的带有完整绝对路径的文件名
          // 使用require.resolve函数查询模块文件名时并不会加载该模块
          delete require.cache[require.resolve(i)]
        }
      })
    }
    
    module.exports = app => {
      // es6 polyfill
      require('@babel/register')
    
      // parse app.body
      // https://expressjs.com/en/4x/api.html#req.body
      app.use(bodyParser.json())
      app.use(bodyParser.urlencoded({
        extended: true
      }))
    
      const mockRoutes = registerRoutes(app)
      var mockRoutesLength = mockRoutes.mockRoutesLength
      var mockStartIndex = mockRoutes.mockStartIndex
    
      // watch files, hot reload mock server
      chokidar.watch(mockDir, {
        ignored: /mock-server/,
        ignoreInitial: true
      }).on('all', (event, path) => {
        if (event === 'change' || event === 'add') {
          try {
            // remove mock routes stack
            app._router.stack.splice(mockStartIndex, mockRoutesLength)
    
            // clear routes cache
            unregisterRoutes()
    
            const mockRoutes = registerRoutes(app)
            mockRoutesLength = mockRoutes.mockRoutesLength
            mockStartIndex = mockRoutes.mockStartIndex
    
            console.log(chalk.magentaBright(`\n > Mock Server hot reload success! changed  ${path}`))
          } catch (error) {
            console.log(chalk.redBright(error))
          }
        }
      })
    }
    
    ~~~

    

  * webpack-dev-serve: vue-cli集成，利用after属性读取mock，模拟REST API

    ~~~js
    // vue.config.js
    devServer: {
        after: require('./mock/mock-server.js')
    }
    ~~~

    

  

#### 菜单 Menu

> 菜单的生成与路由数据息息相关，另外配合UI库的tabs或者tags做页面的tag view，参考 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin) 与 [D2-admin](https://github.com/d2-projects/d2-admin) 两个后台模板。本质递归组件。

#### 错误上报与数据埋点

* error

  * vue自带全局错误函数处理

    ~~~js
    Vue.config.errorHandler = function (err, vm, info) {
      // handle error
      // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
      // 只在 2.2.0+ 可用
    }
    ~~~

  * 组件内错误钩子

    ~~~txt
    errorCaptured
    类型：(err: Error, vm: Component, info: string) => ?boolean
    
    详细：
    
    当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 false 以阻止该错误继续向上传播。
    错误传播规则
    
    默认情况下，如果全局的 config.errorHandler 被定义，所有的错误仍会发送它，因此这些错误仍然会向单一的分析服务的地方进行汇报。
    
    如果一个组件的继承或父级从属链路中存在多个 errorCaptured 钩子，则它们将会被相同的错误逐个唤起。
    
    如果此 errorCaptured 钩子自身抛出了一个错误，则这个新错误和原本被捕获的错误都会发送给全局的 config.errorHandler。
    
    一个 errorCaptured 钩子能够返回 false 以阻止错误继续向上传播。本质上是说“这个错误已经被搞定了且应该被忽略”。它会阻止其它任何会被这个错误唤起的 errorCaptured 钩子和全局的 config.errorHandler
    ~~~

* 数据埋点：就是接口的调用，数据的上报可以先缓存在indexedDB里，批量上报，防止频繁调用堵塞事件队列

#### 功能组件 Functional components

* **link组件**: 根据url选择渲染a标签(外链)还是router-link。

* **redirect与refresh**：参考vue-admi，另外还有auth page与 empty page以及error page也是标配。

* **indexedDB**：[localForage,根据兼容性优雅降级，indexedDB>web SQL > localstorage]([localForage](https://github.com/localForage/localForage)) ,一般用不到indexedDB，业务场景可以考虑体量比较大且基本不变化的数据做缓存，如城市信息。另外就是对用户行为，页面操作需要上报或记录的，可以使用indexedDB暂存，利用定时任务批量上报。D2-admin使用了原生的indexedDB做路由缓存，可以参考。

* **[is-online](https://github.com/sindresorhus/is-online)**:  用户离线判断，放axios拦截器里比较好，或者websocket实时检测。

* **iframe**: vue-admin与D2-admin都有涉及这个，需要可以参考，一般业务比较少见。

* **userAgent**：[ua-parser](https://github.com/faisalman/ua-parser-js)  对用户设备信息的判断，移动端经常要处理，本质就是 navigator.userAgent,利用正则提取关键信息。

  ~~~js
  window.navigator.userAgent
  23:56:59.891 "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36"
  
  //客户端类型
  export const CLIENT = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) ? "IOS" : "Android";
  //是否微信
  export const IS_WECHAT = u.toLowerCase().match(/MicroMessenger/i) == "micromessenger";
  //想用来判断微信PC内置浏览器，window能找到windowswechat字样，mac不知道
  //macintosh为macPC浏览器字样
  export const IS_PC = /macintosh|window|windowswechat/.test(u.toLowerCase());
  ~~~

* **tooltips**：element-ui自带tooltips，但是经常与一些插件不兼容，比如jstree, 可以考虑 [tippy](https://atomiks.github.io/tippyjs/) 这款不错的插件

* **注册全局插件**: [vue-create-api](https://github.com/cube-ui/vue-create-api) 滴滴cube-ui里的注册全局组件的插件，非常nice。

* **skeleton**: 骨架屏貌似就在antd里看到，可以考虑扒下来，[ant design vue](https://vue.ant.design/components/skeleton-cn/) 。

