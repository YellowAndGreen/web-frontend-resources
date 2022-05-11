# docker

## 安装使用

### windows配置阿里云镜像加速

在设置的Docker Engine里加入镜像地址

```
{
  "registry-mirrors": ["https://aa25jngu.mirror.aliyuncs.com"],
  "insecure-registries": [],
  "debug": false,
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "builder": {
    "gc": {
      "enabled": true,
      "defaultKeepStorage": "20GB"
    }
  }
}
```

### 常用命令

1. `启动docker： systemctl start docker`
2. `停止docker： systemctl stop docker`
3. `重启docker： systemctl restart docker`

## 镜像命令

1. 显示所有镜像：`docker images`
2. 下载镜像：`docker pull 镜像名字[:TAG]`
3. 查看镜像/容器/数据卷所占的空间：`docker system df `
4. 删除镜像：`docker rmi 某个XXX镜像名字ID`
5. 删除全部镜像：`docker rmi -f $(docker images -qa)`

### 扩展现有镜像

1. 在容器中添加功能
2. docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

## 容器命令

### 容器运行

1. 创建容器并运行：`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

>  OPTIONS说明（常用）：
>
> 有些是一个减号，有些是两个减号 --name="容器新名字"    为容器指定一个名称；
>
> -d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)； 
>
> -i：以交互模式运行容器，通常与 -t 同时使用；
>
> -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；也即启动交互式容器(前台有伪终端，等待交互)； 
>
> -P: 随机端口映射，大写P-p: 指定端口映射，小写p

例如：`docker run -it -p 9000:9000 --name test ubuntu /bin/bash`

2. 列出所有的容器：`docker ps -a`，不加`a`只显示运行的容器
3. 在容器命令行退出容器
   1. 退出并停止容器：`exit`
   2. 退出不停止容器：`ctrl + p q`
4. 停止容器：`docker stop 容器ID或者容器名`
5. 强制停止容器：`docker kill 容器ID或容器名`
6. 启动已经退出的容器：`docker start 容器ID或者容器名`
7. 重启容器：`docker restart 容器ID或者容器名`
8. 进入正在运行的容器并以命令行交互：`docker exec -it 容器ID /bin/bash`

### 主机和容器的文件拷贝

1. 从主机到容器：`docker cp 要拷贝的文件路径 容器名：要拷贝到容器里面对应的路径`
2. 从容器到主机：`docker cp  容器ID:容器内路径 目的主机路径`

### 导入导出容器

1. 导出容器：`docker export 容器ID > 文件名.tar`
2. 导入容器：`type 文件名| docker import - 用户/镜像名:版本号`

### 容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷  

1. 有容器卷存储功能的容器实例：`docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录      镜像名`
2. 容器2继承容器1的卷规则：`docker run -it  --privileged=true --volumes-from 父类  --name u2 ubuntu`

## DockerFile

1. 运行DockerFile文件：最后一个.表示使用当前目录下的DockerFile

   `docker build -t 镜像名:tag版本号 .`

安装java+vim+ifconfig

```dockerfile
FROM centos:7
MAINTAINER zzyy<zzyybs@126.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

## Docker Compose

Docker-Compose是Docker官方的开源项目，负责实现对Docker容器集群的快速编排。

### **Compose**常用命令

```bash
docker-compose -h              # 查看帮助

docker-compose up              # 启动所有docker-compose服务

docker-compose up -d             # 启动所有docker-compose服务并后台运行

docker-compose down             # 停止并删除容器、网络、卷、镜像。

docker-compose exec  yml里面的服务id         # 进入容器实例内部 docker-compose exec docker-compose.yml文件中写的服务id /bin/bash

docker-compose ps            # 展示当前docker-compose编排过的运行的所有容器

docker-compose top           # 展示当前docker-compose编排过的容器进程

docker-compose logs  yml里面的服务id   # 查看容器输出日志

docker-compose config   # 检查配置

docker-compose config -q # 检查配置，有问题才有输出

docker-compose restart  # 重启服务

docker-compose start   # 启动服务

docker-compose stop    # 停止服务
```



### docker-compose文件

```yaml
version: "3"

networks: # 使用自定义网络，容器之间的IP可以通过容器名获取
  anki_network:
    driver: bridge

services:
  web:
	# 使用本地的dockerfile构建镜像
    build:
      context: .
      dockerfile: ./docker-env/django/Dockerfile
    command: uwsgi --ini config/uwsgi.ini  # 容器启动后启动web服务器
    networks:
      - anki_network
    container_name: web
    expose:
      - "80"


  nginx: 
  	# 使用镜像，若本地没有则会拉取镜像
    image: nginx:stable
    container_name: nginx
    ports:
      - "7777:80"
    restart: always # always表容器运行发生错误时一直重启
    volumes:  # 挂载配置和静态文件
      - ./config/nginx.conf:/etc/nginx/nginx.conf
      - ./flashcards/static:/home/static
    networks:
      - anki_network
    # 这个程序的启动依赖于django后端，会在其启动后再开启nginx
    depends_on:
      - web

```

> uwsgi.ini文件

```ini
[uwsgi]
# variables
projectname = Anki
base = /app   # django应用的路径

# configuration
master = true

pythonpath = %(base)
chdir = %(base)
env = DJANGO_SETTINGS_MODULE=%(projectname).settings
module = %(projectname).wsgi:application
socket  = 0.0.0.0:80   # uwsgi和nginx之间使用socket通信
# http  = 0.0.0.0:80
chmod-socket = 666
```

> nginx.conf配置

```ini
worker_processes 1;

events {
    worker_connections 1024;
}

# the upstream components nginx needs to connect to
http{
    # 下面两行使得CSS加载成功后能够作为一个可执行文件使用
    include mime.types;
    default_type application/octet-stream;
    
    upstream Anki {
        server web:80;
    }

    server {
        listen 80;
        server_name  localhost;
        # 静态文件
        location /static/ {
            alias /home/static/;
        }
        # 将uwsgi作为动态请求的应用
         location / {
            uwsgi_pass Anki;   
            include /etc/nginx/uwsgi_params;
   	    }
    }
}
```



## 网络

1.  查看所有网络：`docker network ls`

2. 查看网络详情：`docker network inspect  XXX网络名字`

3. 创建自定义网络：`docker network create <workname>`

   ```bash
   // 指定这个网络后使用这个网络的容器间可以通过容器名通信
   docker run -d -p 8081:8080 --network zzyy_network  --name tomcat81 billygoo/tomcat8-jdk8
   ```

###  网络模式

默认使用的是bridge模式的网络

1. bridge模式：使用--network  bridge指定，默认使用docker0

> bridge模式会为每个容器创建虚拟的ip和端口，类似于NAT

2. host模式：使用--network host指定

> host模式使用主机自身的ip和端口，容器内服务的端口就是主机端口访问地址
>
> **不需要指定端口号**

3. none模式：使用--network none指定

> 不为Docker容器进行任何网络配置，需要自己配置

4. container模式：使用--network container:NAME或者容器ID指定

> 新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。
>
> **由于多个容器用一个IP，所以容器内部的端口不能冲突！！！**

### 查看容器网络IP

使用`docker inspect 容器id| grep "IPAddress"`查看某个容器的虚拟ip

## 常用安装

### mysql

1. 安装：`docker pull mysql:5.7`

2. 运行容器：

```
docker run -d -p 3306:3306 --privileged=true -v /zzyyuse/mysql/log:/var/log/mysql -v /zzyyuse/mysql/data:/var/lib/mysql -v /zzyyuse/mysql/conf:/etc/mysql/conf -e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:5.7
```

自己的实例：

```
docker run -d -p 3307:3306 --privileged=true -e MYSQL_ROOT_PASSWORD=123456 -v C:\Users\Administrator\Desktop\dockerfiles:/etc/mysql --name mysql mysql:5.7
```



3. 修改/etc/mysql中的my.cnf（先运行，再修改，否则文件会变成空）

```
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```



### Redis

1. 运行并修改conf，最后的`-d redis:6.0.8 redis-server /etc/redis/redis.conf`是运行指令

```
docker run  -p 6379:6379 --name myr3 --privileged=true -v /app/redis/redis.conf:/etc/redis/redis.conf -v /app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
```



### MongoDB

1. 下载镜像并运行：`docker run -d --name mongodb -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password -p 27017:27017 mongo:4.4.3`

## 修改Windows下的默认镜像存放路径

1. 删除所有数据：`purge data`

2. `wsl -l -v --all`查询状态，必须都关闭

3. 导出wsl子系统镜像，第二个路径是要放的路径：

   ```bash
   wsl --export docker-desktop D:\Docker\docker-desktop.tar
   wsl --export docker-desktop-data D:\Docker\docker-desktop-data.tar
   ```
   
4. 删除现有的wsl子系统：

   ```bash
   wsl --unregister docker-desktop
   wsl --unregister docker-desktop-data
   ```

5. 重新创建wsl子系统：

   ```bash
   wsl --import docker-desktop D:\Docker\docker-desktop D:\Docker\docker-desktop.tar --version 2
   
   wsl --import docker-desktop-data D:\Docker\docker-desktop-data D:\Docker\docker-desktop-data.tar --version 2
   ```

   

