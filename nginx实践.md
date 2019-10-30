



## 静态资源WEB服务

* ![CDN](C:\Users\卿松\Desktop\图片\nginx静态服务1.png)

* 相关模块

  * ![sendfile](C:\Users\卿松\Desktop\图片\nginx静态服务读取文件.png 'sendfile')
  * ![传输效率](C:\Users\卿松\Desktop\图片\nginx静态服务传输效率.png '传输效率')
  * ![传输实时性](C:\Users\卿松\Desktop\图片\nginx静态服务传输实时性.png '传输实时性')
  * ![压缩](C:\Users\卿松\Desktop\图片\nginx静态服务压缩.png '压缩') 
  * ![压缩比](C:\Users\卿松\Desktop\图片\nginx静态服务压缩比.png)

* 场景

  * ![](C:\Users\卿松\Desktop\图片\nginx静态服务场景1.png)

* 缓存

  * ![](C:\Users\卿松\Desktop\图片\nginx资源服务缓存原理.png)
  * ![](C:\Users\卿松\Desktop\图片\nginx资源服务配置语法.png)
  * ![](C:\Users\卿松\Desktop\图片\nginx静态资源缓存文件.png)

* 跨站访问

  * ![](C:\Users\卿松\Desktop\图片\nginx静态服务跨站语法.png)
  * ![](C:\Users\卿松\Desktop\图片\nginx静态服务跨站访问配置.png)

* 防盗链

  * ![](C:\Users\卿松\Desktop\图片\nginx防盗链语法.png)

    ![nginx静态服务防盗链配置](C:\Users\卿松\Desktop\图片\nginx静态服务防盗链配置.png)



## 代理服务

* **正向代理**

  * ![](C:\Users\卿松\Desktop\图片\正向代理.png)
  * ![](C:\Users\卿松\Desktop\图片\正向代理配置.png)
  * 

* **反向代理**

  * ![](C:\Users\卿松\Desktop\图片\反向代理.png)
  * ![](C:\Users\卿松\Desktop\图片\反向代理1.png)
  * ![](C:\Users\卿松\Desktop\图片\代理语法.png)
  * ![](C:\Users\卿松\Desktop\图片\反向代理简单配置.png)

* **补充语法**

  * ![](C:\Users\卿松\Desktop\图片\代理补充语法-超时.png)

    ![代理补充语法-缓存区](C:\Users\卿松\Desktop\图片\代理补充语法-缓存区.png)

    ![代理补充语法-头信息](C:\Users\卿松\Desktop\图片\代理补充语法-头信息.png)

    ![代理补充语法-重定向](C:\Users\卿松\Desktop\图片\代理补充语法-重定向.png)

  * ![](C:\Users\卿松\Desktop\图片\代理补充语法配置.png)

  * 



## 负载均衡调度器SLB

* 

## 动态缓存

* 语法

  ![](C:\Users\卿松\Desktop\图片\nginx缓存语法1.png)

  ![nginx缓存语法2](C:\Users\卿松\Desktop\图片\nginx缓存语法2.png)

  ![nginx缓存语法3](C:\Users\卿松\Desktop\图片\nginx缓存语法3.png)

  ![nginx缓存语法4](C:\Users\卿松\Desktop\图片\nginx缓存语法4.png)

  

  ![](C:\Users\卿松\Desktop\图片\nginx缓存配置.png)

* 补充

  * ![](C:\Users\卿松\Desktop\图片\nginx缓存补充-不缓存.png)

    ![nginx缓存补充-不缓存配置](C:\Users\卿松\Desktop\图片\nginx缓存补充-不缓存配置.png)

    ![nginx缓存补充-清理](C:\Users\卿松\Desktop\图片\nginx缓存补充-清理.png)

* 缓存命中分析

  * ![](C:\Users\卿松\Desktop\图片\nginx缓存命中率.png)

    ![缓存命中-upstream-status](C:\Users\卿松\Desktop\图片\缓存命中-upstream-status.png)

    ![缓存命中分析](C:\Users\卿松\Desktop\图片\缓存命中分析.png)

    ![](C:\Users\卿松\Desktop\图片\缓存命中分析1.png)

    ![](C:\Users\卿松\Desktop\图片\缓存命中分析2.png)![](C:\Users\卿松\Desktop\图片\缓存命中分析3.png)$NF==那有错误，后面的要用引号包裹

* 分片请求

  * ![](C:\Users\卿松\Desktop\图片\缓存分片请求.png)

    ![缓存分片请求1](C:\Users\卿松\Desktop\图片\缓存分片请求1.png)

    ![缓存分片请求2](C:\Users\卿松\Desktop\图片\缓存分片请求2.png)

