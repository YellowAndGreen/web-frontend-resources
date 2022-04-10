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



## 镜像命令

### 扩展现有镜像

1. 在容器中添加功能
2. docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

## 容器命令

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

   

