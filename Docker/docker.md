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

### 容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷  

1. 有容器卷存储功能的容器实例：`docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录      镜像名`
2. 容器2继承容器1的卷规则：`docker run -it  --privileged=true --volumes-from 父类  --name u2 ubuntu`



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

