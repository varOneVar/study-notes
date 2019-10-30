#### 镜像 image

* 类比 **类** 

* 镜像是一个联合只读文件夹，由很多只读层构成，每次更新会生成新的镜像层，多次在同一个镜像上更新，会造成镜像变大，增量式

* 当内存快满时，docker会kill占量最大的镜像，称为OOM，如果页面经常挂壁，就是OOM了

* 指令

  ~~~shell
  # 查看所有镜像 ,后面的是过滤查询某镜像
  docker images | grep container-name
  # 拉取镜像 默认使用latest标签
  docker pull image_name:tag
  # 搜索镜像仓库docker hub里的镜像
  docker search image_name
  # 获取镜像详细信息
  docker inspect image_name or container_name or id
  # 删除,支持多删
  docker rmi image_name or image_id ...
  # 镜像迁移， save将镜像内容输出 >接收, -o可以指定输出文件，支持多个批量
  docker save webapp:1.0 > webapp-1.0.tar
  docker save -o ./webapp-1.0.tar webapp:1.0
  docker save -o ./images.tar webapp:1.0 nginx:1.12 mysql:5.7
  # 镜像导入, < 导入或 -i 指定输入文件
  docker load < webapp-1.0.tar
  docker load -i webapp-1.0.tar
  ~~~

* 镜像仓库

  * 测试环境将镜像推向镜像仓库，正式环境拉取镜像仓库的镜像，类比git
  * [docker hub](hub.docker.com/)

#### 容器container

* 类比 **类的实例** 

* 在镜像上生成一个可读写的镜像层，相当于活动空间

* 由 镜像 + 程序运行环境 + 指令集 组成

* 指令

  ~~~shell
  # 根据所给镜像创建容器,--name为容器创建名字
  docker create image_name --name container_name
  # 启动容器
  docker start container_name
  # docker create 与 docker start合并, -d表示进入后台运行
  docker run --name container_name -d imager_name
  # 查看容器, -a是所有，默认是是运行中的容器，created，up，exitd三种状态
  docker ps -a
  # 停止容器
  docker stop container_name
  # 删除容器
  docker rm container_name
  # 指向命令
  docker exec command
  # 执行容器中的bash,-i是保持输入流，-t表示启用伪终端，形成bash交互
  docker exec -it container_name bash
  # 提交容器更改持久化为镜像层
  docker commit -m "注释" contaienr_name
  # 镜像命名或改名
  docker tag image_id container_name:name [container_name:old_name]
  # 容器导出
  docker export -o ./webapp.tar webapp
  # 容器导入，以镜像形式导入，本质镜像
  docker import ./webapp.tar webapp:1.0
  ~~~

  

#### 网络network

* 核心

  * 沙盒，虚拟网络栈
  * 网络，虚拟子网
  * 端点，容器或网络隔离墙的通信桥梁

* 例子

  * 创建一个 MySQL 容器，将运行我们 Web 应用的容器连接到这个 MySQL 容器上

    ~~~shell
    $ sudo docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql
    $ sudo docker run -d --name webapp --link mysql webapp:latest
    # 假设Web 应用中使用的是 JDBC 进行数据库连接的,将容器的网络命名填入到连接地址中
    String url = "jdbc:mysql://mysql:3306/webapp";
    ~~~

  * 暴露端口

    ~~~shell
    # --expose暴露，docker ps查看
    docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes --expose 13306 --expose 23306 mysql:5.7
    ~~~

  * 设置别名

    ~~~shell
    # --link <name>:<alias>
    docker run -d --name webapp --link mysql:database webapp:latest
    # Web 应用中使用 MySQL 连接时，我们就可以使用 database 来代替连接地址
    String url = "jdbc:mysql://database:3306/webapp";
    ~~~

  * 创建网络

    ~~~shell
    # 创建网络, -d选择网络驱动类型， bridge(默认), host, overlay,maclan,null(none)
    docker network create -d bridge name
    # 查看已经存在的网络
    docker network ls
    # --network 可以在运行时加入这个指令添加网络
    ~~~

  * 端口映射

    ~~~shell
    # -p 映射端口，外部访问发送到关联的容器端口,format：-p <ip>:<host-port>:<container-port>， ip默认0.0.0.0监听所有网卡
    docker run -d --name nginx -p 80:80 -p 443:443 nginx:1.12
    ~~~

    

#### 数据卷 volume

* 挂载目录文件

  ```shell
  # -v <host-path>:<container-path>
  docker run -d --name nginx -v /webapp/html:/usr/share/nginx/html nginx:1.12
  ```

#### Dockerfile

~~~dockerfile
FROM debian:stretch-slim

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r redis && useradd -r -g redis redis

# grab gosu for easy step-down from root
# https://github.com/tianon/gosu/releases
ENV GOSU_VERSION 1.10
RUN set -ex; \
	\
	fetchDeps=" \
		ca-certificates \
		dirmngr \
		gnupg \
		wget \
	"; \
	apt-get update; \
	apt-get install -y --no-install-recommends $fetchDeps; \
	rm -rf /var/lib/apt/lists/*; \
	\
	dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
	wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
	wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
	gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
	gpgconf --kill all; \
	rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc; \
	chmod +x /usr/local/bin/gosu; \
	gosu nobody true; \
	\
	apt-get purge -y --auto-remove $fetchDeps

ENV REDIS_VERSION 3.2.12
ENV REDIS_DOWNLOAD_URL http://download.redis.io/releases/redis-3.2.12.tar.gz
ENV REDIS_DOWNLOAD_SHA 98c4254ae1be4e452aa7884245471501c9aa657993e0318d88f048093e7f88fd

# for redis-sentinel see: http://redis.io/topics/sentinel
RUN set -ex; \
	\
	buildDeps=' \
		wget \
		\
		gcc \
		libc6-dev \
		make \
	'; \
	apt-get update; \
	apt-get install -y $buildDeps --no-install-recommends; \
	rm -rf /var/lib/apt/lists/*; \
	\
	wget -O redis.tar.gz "$REDIS_DOWNLOAD_URL"; \
	echo "$REDIS_DOWNLOAD_SHA *redis.tar.gz" | sha256sum -c -; \
	mkdir -p /usr/src/redis; \
	tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1; \
	rm redis.tar.gz; \
	\
# disable Redis protected mode [1] as it is unnecessary in context of Docker
# (ports are not automatically exposed when running inside Docker, but rather explicitly by specifying -p / -P)
# [1]: https://github.com/antirez/redis/commit/edd4d555df57dc84265fdfb4ef59a4678832f6da
	grep -q '^#define CONFIG_DEFAULT_PROTECTED_MODE 1$' /usr/src/redis/src/server.h; \
	sed -ri 's!^(#define CONFIG_DEFAULT_PROTECTED_MODE) 1$!\1 0!' /usr/src/redis/src/server.h; \
	grep -q '^#define CONFIG_DEFAULT_PROTECTED_MODE 0$' /usr/src/redis/src/server.h; \
# for future reference, we modify this directly in the source instead of just supplying a default configuration flag because apparently "if you specify any argument to redis-server, [it assumes] you are going to specify everything"
# see also https://github.com/docker-library/redis/issues/4#issuecomment-50780840
# (more exactly, this makes sure the default behavior of "save on SIGTERM" stays functional by default)
	\
	make -C /usr/src/redis -j "$(nproc)"; \
	make -C /usr/src/redis install; \
	\
	rm -r /usr/src/redis; \
	\
	apt-get purge -y --auto-remove $buildDeps

RUN mkdir /data && chown redis:redis /data
VOLUME /data
WORKDIR /data

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD ["redis-server"]
~~~

* 指令介绍

  ~~~doc
  # 指定基础镜像,三种形式， 第一条指令必须是FROM指令
  FROM：
      FROM <image> [AS <name>]
      FROM <image>[:<tag>] [AS <name>]
      FROM <image>[@<digest>] [AS <name>]
  
  # 向控制台发送命令，支持 \ 分割命令换行
  RUN：
  	RUN <command>
  	RUN ["executable", "param1", "param2"]
  	
  # 启动进程号为1的进程，镜像与此进程共存亡
  # ENTRYPOINT 与 CMD 同时给出时，CMD 中的内容会作为 ENTRYPOINT 定义命令的参数，最终执行容器启动的还是 ENTRYPOINT 中给出的命令
  # ENTRYPOINT优先级大过CMD
  # ENTRYPOINT 指令主要用于对容器进行一些初始化，而 CMD 指令则用于真正定义容器中主程序的启动命令。
  ENTRYPOINT 和 CMD：
  	ENTRYPOINT ["executable", "param1", "param2"]
      ENTRYPOINT command param1 param2
  
      CMD ["executable","param1","param2"]
      CMD ["param1","param2"]
      CMD command param1 param2
      
  # 为镜像指定要暴露的端口
  EXPOSE：
  	EXPOSE <port> [<port>/<protocol>...]
  
  #  VOLUME 指令定义基于此镜像的容器所自动建立的数据卷
  VOLUME：
  	VOLUME ["/data"]
  	
  # COPY 或 ADD 指令能够帮助我们直接从宿主机的文件系统里拷贝内容到镜像里的文件系统中。
  # ADD 能够支持使用网络端的 URL 地址作为 src 源，并且在源文件被识别为压缩包时，自动进行解压
  COPY 和 ADD：
  	COPY [--chown=<user>:<group>] <src>... <dest>
      ADD [--chown=<user>:<group>] <src>... <dest>
  
      COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
      ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
      
  
  ~~~

* 构建镜像

  ~~~shell
  # 接收一个目录路径做环境目录参数，默认在此目录寻找Dockerfile文件， -f指定Dockerfile文件路径， -t 指定新生成镜像名称
  docker build -t webapp:latest -f ./webapp/a.Dockerfile ./webapp
  ~~~

* 技巧

  ~~~dockerfile
  # ARG 建立参数变量
  FROM debian:stretch-slim
  
  ## ......
  
  ARG TOMCAT_MAJOR
  ARG TOMCAT_VERSION
  
  ## ......
  # 将 Tomcat 的版本号通过 ARG 指令定义为参数变量，在调用下载 Tomcat 包时，使用变量替换掉下载地址中的版本号
  RUN wget -O tomcat.tar.gz "https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz"
  
  ## ......
  
  #最后在构建命令中通过 --build-arg 设置变量值
  docker build --build-arg TOMCAT_MAJOR=8 --build-arg TOMCAT_VERSION=8.0.53 -t tomcat:8.0 ./tomcat
  ~~~

  ~~~dockerfile
  # ENV 环境变量，值直接定义
  FROM debian:stretch-slim
  
  ## ......
  
  ENV TOMCAT_MAJOR 8
  ENV TOMCAT_VERSION 8.0.53
  
  ## ......
  
  RUN wget -O tomcat.tar.gz "https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-$TOMCAT_MAJOR/v$TOMCAT_VERSION/bin/apache-tomcat-$TOMCAT_VERSION.tar.gz"
  
  # 构建命令中通过-e或--env覆盖Dockerfile中的值
  docker run -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:5.7
  # ENV优先权大过ARG
  # 多用 \ 合并命令，因为镜像是增量式的
  RUN apt-get update; \
      apt-get install -y --no-install-recommends $fetchDeps; \
      rm -rf /var/lib/apt/lists/*;
  ~~~


#### docker compose

* 安装

  ~~~shell
  $ sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  $ sudo chmod +x /usr/local/bin/docker-compose
  $ sudo docker-compose version
  ~~~

* 配置docker-compose.yml

  ~~~yml
  # docker-compose的版本
  version: '3'
  
  services:
  
    redis:
      image: redis:3.2
      networks:
        - backend
      volumes:
        - ./redis/redis.conf:/etc/redis.conf:ro
      ports:
        - "6379:6379"
      command: ["redis-server", "/etc/redis.conf"]
  
    database:
      image: mysql:5.7
      networks:
        backend:
          aliases:
            - backend.database
      volumes:
        - ./mysql/my.cnf:/etc/mysql/my.cnf:ro
        - mysql-data:/var/lib/mysql
      environment:
        - MYSQL_ROOT_PASSWORD=my-secret-pw
      ports:
        - "3306:3306"
  
    webapp:
      build:
        context: ./webapp
        dockerfile: webapp-dockerfile
        args:
          - JAVA_VERSION=1.6
      networks:
        backend:
          aliases:
            - backend.webapp
        frontend:
          aliases:
            - frontend.webapp
      volumes:
        - ./webapp:/webapp
      # 定义依赖启动顺序
      depends_on:
        - redis
        - database
  
    nginx:
      image: nginx:1.12
      networks:
        - frontend
      volumes:
        - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
        - ./nginx/conf.d:/etc/nginx/conf.d:ro
        - ./webapp/html:/webapp/html
      depends_on:
        - webapp
      # 端口用引号包裹避免被当作时间解析
      ports:
        - "80:80"
        - "443:443"
  
  networks:
    frontend:
      driver: bridge
      ipam:
        driver: default
        config:
          - subnet: 10.10.1.0/24
    backend:
  
  volumes:
    mysql-data:
    	# Docker Compose 在创建项目时不会直接创建数据卷，而是优先从 Docker Engine 中已有的数据卷里寻找并直接采用
    	external: true
  ~~~

* 指令

  ~~~shell
  # 启动 -d为后台运行， 关闭就是 up -> down, -f可以修改配置文件路径， -p为项目命名
  docker-compose -f ./compose/docker-compose.yml -p myapp up -d
  # 查看实时日志，与docker无异
  docker-compose logs xxx
  # 其他类比， up相当于docker run
  $ sudo docker-compose create webapp
  $ sudo docker-compose start webapp
  $ sudo docker-compose stop webapp
  ~~~

  



#### 安装

~~~shell
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
$
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce
$
# docker服务开机自启动
$ sudo systemctl enable docker
# docker服务启动
$ sudo systemctl start docker
# 重新启动
$ sudo systemctl restart docker

# 镜像源配置 /etc/docker/daemon.json
# https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
{
    "registry-mirrors": [
        "https://registry.docker-cn.com"
    ]
}
~~~

#### 指令

~~~shell
# 查看docker engine
docker info
~~~



#### 磁盘占满

* 命令清除

  ~~~shell
  # df -i 查看inode空间
  # df -h 查看磁盘空间 --max-depth=1 |sort，最大深度1，排序
  # du -sh * 查看各文件内存占比 *代表所有文件
  # docker system df -v 查看docker镜像容器数据卷占内存大小,-v增加细节
  # 清除没打标签的镜像
  docker rmi $(docker images -q -f "dangling=true")
  # 清除退出的容器
  docker rm -v $(docker ps -a -q -f status=exited)
  # 清除所有未使用镜像，容器，数据卷， -a 清除了未使用的镜像，不加只会清除悬空(无名称)镜像
  docker system prune -a
  # 删除所有未被任何容器关联引用的卷：
  docker volume rm $(docker volume ls -qf dangling=true)
  # 也可以直接使用如下指令，删除所有未被任何容器关联引用的卷（但建议使用上面的方式）
  # 【说明】轮询到还在使用的卷时，会有类似"volume is in use"的告警信息，所以相关卷不会被删除，忽略即可。
  docker volume rm $(docker volume ls -q)
  # 删除所有已退出的容器
  docker rm -v $(docker ps -aq -f status=exited)
  # 删除所有状态为 dead 的容器
  docker rm -v $(docker ps -aq -f status=dead)
  
  # 主要问题还是/var/lib/docker占满导致docker根目录(/dev/vda1,2)内存爆满，需要备份镜像，迁移到空文件夹里
  ~~~

* 日志占内存问题

  ~~~shell
  # /etc/docker/daemon.json 全局配置日志文件最大size
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "10m",
      "max-file": "3",
      "labels": "production_status",
      "env": "os,customer"
    }
  }
  # 命令参数 启动docker的时候 --log-opt max-size=50 命令配置最大size
  ~~~

* 镜像迁移

  ~~~shell
  # 停止docker
  systemctl stop docker
  # 1.找个空间大的目录,df -h查看一下
  # rsync [OPTION...] SRC... [DEST]，rsync（remote synchronize）是一个远程数据同步工具
  # -avz: a是归档模式，使用递归 v是详细输出模式， z是传输时压缩处理
  mkdir -p /home/docker/lib
  rsync -avz /var/lib/docker/ /home/docker/lib/
  
  # 2.配置 /etc/systemd/system/docker.service.d/devicemapper.conf。查看 devicemapper.conf 是否存在。如果不存在，就新建
  mkdir -p /etc/systemd/system/docker.service.d/
  vi /etc/systemd/system/docker.service.d/devicemapper.conf
  # 写入
  [Service]
  ExecStart=
  ExecStart=/usr/bin/dockerd --graph=/home/docker/lib/docker
  
  # 3.重新加载
  systemctl daemon-reload
  systemctl restart docker
  systemctl enable docker
  
  # 4.确保顺利运行docker info，检查信息里根目录是否被修改
  ...
  Docker Root Dir: /home/docker/lib/docker
  Debug Mode (client): false
  Debug Mode (server): false
  Registry: https://index.docker.io/v1/
  ...
  
  # 5. 运行docker images确认镜像还在，然后再删了原来的/var/lib/docker
  
  # 6. 如果启动错误Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
  # 在 /etc/docker 目录下创建daemon.json文件，并且加入以下配置
  touch daemon.json
  vi daemon.json
  {
    "storage-driver": "overlay2",
    "storage-opts": [
      "overlay2.override_kernel_check=true" 
    ]
  }
  # 再次启动
  systemctl start docker
  
  # 如果镜像运行错误：WARNING: IPv4 forwarding is disabled. Networking will not work.
  # /usr/bin/docker-current: Error response from daemon: shim error: docker-runc not installed on system.
  cd /usr/libexec/docker/
  ln -s docker-runc-current docker-runc
  
  
  ## 检查是不是其他进程的日志满了
  # 查看进程pid
  lsof /var/log/xxx
  # 查看进程工作目录
  pwdx 进程pid
  # 查看进程程序状态
  ps -aux | grep 进程pid
  ~~~

  

  

