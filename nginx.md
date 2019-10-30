

### 目录

![日志切割](C:\Users\卿松\Desktop\图片\nginx1.png)

![](C:\Users\卿松\Desktop\图片\nginx2.png)

![nginx3](C:\Users\卿松\Desktop\图片\nginx3.png)

![nginx4](C:\Users\卿松\Desktop\图片\nginx4.png)

![nginx5](C:\Users\卿松\Desktop\图片\nginx5.png)

![nginx6](C:\Users\卿松\Desktop\图片\nginx6.png)

### 语法

![](C:\Users\卿松\Desktop\图片\nginx7.png)

![nginx8](C:\Users\卿松\Desktop\图片\nginx8.png)

### 多服务配置

* 多 ip
* 多端口
* 多域名
  * 修改配置里的server_name

### 请求限制

* 连接频率限制 - limit_conn_module

  * ![连接限制](C:\Users\卿松\Desktop\图片\nginx限制.png)

* 请求频率限制 - limit_req_module

  * ![请求限制](C:\Users\卿松\Desktop\图片\nginx限制1.png)

  * burst代表延迟到下一次请求

  * ![](C:\Users\卿松\Desktop\图片\nginx限制2.png)

    

### 访问控制

* 基于IP的访问控制 - http_access_module

  * ![](C:\Users\卿松\Desktop\图片\nginx访问控制1.png)
  * 
  * ![](C:\Users\卿松\Desktop\图片\nginxip控制访问.png)
  * ![ip限制局限性处理](C:\Users\卿松\Desktop\图片\ngixn访问控制2.png)
  * ![](C:\Users\卿松\Desktop\图片\http_x_forwarded_for.png)

* 基于用户的信任登陆 - http_auth_basic_module

  * ![](C:\Users\卿松\Desktop\图片\nginx访问控制3.png)

  * ![](C:\Users\卿松\Desktop\图片\nginx访问控制4.png)

    

  