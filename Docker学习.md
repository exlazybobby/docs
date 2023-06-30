```shell
# M1 数据库8.0
docker container run --name mysql -e MYSQL_ROOT_PASSWORD=root -p3306:3306 -d -it --restart=always -v mysql-data:/var/lib/mysql mysql:oracle

CREATE USER `lixing`@`%` IDENTIFIED BY 'xing.li@2021';

GRANT Alter, Alter Routine, Create, Create Routine, Create Temporary Tables, 
Create View, Index, Insert, Lock Tables, Process, 
References, Reload, Select, Show Databases, Show View, Update 
ON *.* TO `lixing`@`%`;

flush privileges;

all privileges：所有权限。
select：读取权限。
create：创建权限。
delete：删除权限。
update：更新权限。
drop：删除数据库、数据表权限。

docker run -itd --restart=always --name redis -p 6379:6379 redis

# centos add user
adduser xubin
passwd xubin
chmod -v u+w /etc/sudoers
vi /etc/sudoers
root    ALL=(ALL)       ALL
xubin    ALL=(ALL)       ALL （添加这一行）
chmod -v u-w /etc/sudoers



docker run -d --name nps --net=host -v /home/xubin/nps/conf:/conf ffdfgdfg/nps restart=always

```





## linux安装docker

```shell
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```



```shell
# 创建并运行,没有镜像的话会拉取
docker container run nginx

# 当前正在运行的容器
docker container ls

# 停止某个容器
docker container stop <容器id>

# 显示运行中的容器状态
docker container ps

# 显示所有的容器状态
docker container ls -a

# 删除容器image
docker container rm <Image ID>
```

## Docker命令小技巧

```shell
# 停止指定容器(多个)
dockr sop <容器id> <容器id> <容器id>
docker container ps -a
# 列出所有正在运行的容器id
docker container ps -aq
# 停止运行中的所有容器
docker container stop $(docker container ps -aq)
# 停止用stop，删除用rm
# 不能删除正在运行的容器
docker container stop <容器id>
docker container rm <容器id>
# 强制删除运行中的容器
docker container rm -f <容器id> 
# 清理后台停止的容器
docker system prune -f
# 清理不用的镜像
docker image prune -a
# 容器不运行时删除容器
docker container run  --rm -d -it nginx
```

## Docker 参数使用

```shell
# 将容器内部的端口映射到外部
docker container run -p 80:80 nginx

# 容器后台运行 -d (--detach)
docker container run -d -p 80:80 nginx

# 进入容器看看log
docker container attach <容器id>
docker container logs <容器id>
# 动态显示log
docker container logs -f <容器id>

# 创建交互式容器 （连接容器的shell）
docker container run -it ubuntu sh

# 交互式进入容器（在一个已经运行的容器里执行一个额外的command）
docker container exec -it <容器id> sh

```

## Docker进程

- 容器实际上就是个进程

  ```shell
  # 可以查看到进程和子进程，在宿主机上能查到相关的进程
  docker container top <容器id>
  ```

  

## 镜像获取

- pull from `registry` (online) 从registry拉取
  - public（公有）
  - private（私有）
- build from `Dockerfile` (online) 从Dockerfile构建
- load from `file` (offline) 文件导入 （离线）

##### 从registry拉取

```shell
docker image pull nginx
docker image search nginx
docker image pull quay.io/bitnami/nginx

# 显示镜像更多信息
docker image inspect <Image id>

# 删除镜像
docker image rm <Image id>

# 正在运行中的镜像无法删除
docker container stop <容器id>
docker container rm <容器id>
docker image rm <Image id>
```

##### 导出/导入镜像

```shell
# 导出镜像成文件
docker image save quay.io/bitnami/nginx -o nginx.image
# 倒入镜像文件
docker iamge load -i nginx.image
```

##### Dockerfile介绍

- Dockerfile是用于构建docker镜像的文件
- Dockerfile里包含了构建镜像所需的“指令”
- Dockerfile有其特定的语法规则

Dockerfile语法：https://docs.docker.com/engine/reference/builder/

## 举例：执行一个Python程序

容器及进程，所以镜像就是一个运行这个进程所需要的环境。

假如我们要在一台ubuntu 21.04上运行下面这个hello.py的Python程序

hello.py的文件内容：

```shell
print("hello docker")
```

第一步，准备Python环境(在容器内执行)

```shell
apt-get update && \
DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y python3.9 python3-pip python3.9-dev
```

第一步，运行hello.py

```shell
$ python3 hello.py
hello docker
```

------

## 一个Dockerfile的基本结构

Dockerfile

```dockerfile
FROM ubuntu:21.04
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y python3.9 python3-pip python3.9-dev
ADD hello.py /
CMD ["python3", "/hello.py"]
```

```shell
# 通过Dockerfile构建容器
docker image build -t hello:1.0.0 .
```

```shell
# 修改image的tag
docker image tag <old tag> <new tag>
```

##### 将image推送到dockerhub

```shell
docker login
# 修改镜像名字注：必须以 用户名/镜像名）
docker image tag nginx:1.0 exlobin/nginx:1.0
# 推送
docker push exlobin/nginx:1.0
# 拉取(注：要加上tag)
docker pull docker pull exlobin/nginx:1.0
```

##### 通过容器commit一个image

```shell
docker container commit 3f3 exlobin/nginx
docker push exlobin/nginx
```

#### Dockerfile完全指南

# 通过 RUN 执行指令

`RUN` 主要用于在Image里执行指令，比如安装软件，下载文件等。

```
$ apt-get update
$ apt-get install wget
$ wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz
$ tar zxf ipinfo_2.0.1_linux_amd64.tar.gz
$ mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo
$ rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

## Dockerfile

```
FROM ubuntu:21.04
RUN apt-get update
RUN apt-get install -y wget
RUN wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz
RUN tar zxf ipinfo_2.0.1_linux_amd64.tar.gz
RUN mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo
RUN rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

镜像的大小和分层

```shell
$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ipinfo       latest    97bb429363fb   4 minutes ago   138MB
ubuntu       21.04     478aa0080b60   4 days ago      74.1MB
$ docker image history 97b
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
97bb429363fb   4 minutes ago   RUN /bin/sh -c rm -rf ipinfo_2.0.1_linux_amd…   0B        buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c mv ipinfo_2.0.1_linux_amd64 /…   9.36MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c tar zxf ipinfo_2.0.1_linux_am…   9.36MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c wget https://github.com/ipinf…   4.85MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c apt-get install -y wget # bui…   7.58MB    buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c apt-get update # buildkit        33MB      buildkit.dockerfile.v0
<missing>      4 days ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      4 days ago      /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>      4 days ago      /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B
<missing>      4 days ago      /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>      4 days ago      /bin/sh -c #(nop) ADD file:d6b6ba642344138dc…   74.1MB
```

每一行的RUN命令都会产生一层image layer, 导致镜像的臃肿。

## 改进版Dockerfile

```
FROM ubuntu:21.04
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz && \
    tar zxf ipinfo_2.0.1_linux_amd64.tar.gz && \
    mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ipinfo-new   latest    fe551bc26b92   5 seconds ago    124MB
ipinfo       latest    97bb429363fb   16 minutes ago   138MB
ubuntu       21.04     478aa0080b60   4 days ago       74.1MB
$ docker image history fe5
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
fe551bc26b92   16 seconds ago   RUN /bin/sh -c apt-get update &&     apt-get…   49.9MB    buildkit.dockerfile.v0
<missing>      4 days ago       /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      4 days ago       /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>      4 days ago       /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B
<missing>      4 days ago       /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>      4 days ago       /bin/sh -c #(nop) ADD file:d6b6ba642344138dc…   74.1MB
$
```

# 文件复制和目录操作 (ADD,COPY,WORKDIR)

往镜像里复制文件有两种方式，`COPY` 和 `ADD` , 我们来看一下两者的不同。

## 复制普通文件

`COPY` 和 `ADD` 都可以把local的一个文件复制到镜像里，如果目标目录不存在，则会自动创建

```dockerfile
FROM python:3.9.5-alpine3.13
COPY hello.py /app/hello.py
```

比如把本地的 hello.py 复制到 /app 目录下。 /app这个folder不存在，则会自动创建

## 复制压缩文件

`ADD` 比 COPY高级一点的地方就是，如果复制的是一个gzip等压缩文件时，ADD会帮助我们自动去解压缩文件。

```dockerfile
FROM python:3.9.5-alpine3.13
ADD hello.tar.gz /app/
```

## 如何选择

因此在 COPY 和 ADD 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD。



# 构建参数和环境变量 (ARG vs ENV)

`ARG` 和 `ENV` 是经常容易被混淆的两个Dockerfile的语法，都可以用来设置一个“变量”。 但实际上两者有很多的不同。

```
FROM ubuntu:21.04
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-2.0.1/ipinfo_2.0.1_linux_amd64.tar.gz && \
    tar zxf ipinfo_2.0.1_linux_amd64.tar.gz && \
    mv ipinfo_2.0.1_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_2.0.1_linux_amd64.tar.gz
```

## ENV

```
FROM ubuntu:21.04
ENV VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
```

## ARG

```
FROM ubuntu:21.04
ARG VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
```

## 区别

![docker-image](https://dockertips.readthedocs.io/en/latest/_images/docker_environment_build_args.png)

ARG 可以在镜像build的时候动态修改value, 通过 `--build-arg`

```
$ docker image build -f .\Dockerfile-arg -t ipinfo-arg-2.0.0 --build-arg VERSION=2.0.0 .
$ docker image ls
REPOSITORY         TAG       IMAGE ID       CREATED          SIZE
ipinfo-arg-2.0.0   latest    0d9c964947e2   6 seconds ago    124MB
$ docker container run -it ipinfo-arg-2.0.0
root@b64285579756:/#
root@b64285579756:/# ipinfo version
2.0.0
root@b64285579756:/#
```

ENV 设置的变量可以在Image中保持，并在容器中的环境变量里

# 容器启动命令 CMD

CMD可以用来设置容器启动时默认会执行的命令。

- 容器启动时默认执行的命令
- 如果docker container run启动容器时指定了其它命令，则CMD命令会被忽略
- 如果定义了多个CMD，只有最后一个会被执行。

```
FROM ubuntu:21.04
ENV VERSION=2.0.1
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/ipinfo/cli/releases/download/ipinfo-${VERSION}/ipinfo_${VERSION}_linux_amd64.tar.gz && \
    tar zxf ipinfo_${VERSION}_linux_amd64.tar.gz && \
    mv ipinfo_${VERSION}_linux_amd64 /usr/bin/ipinfo && \
    rm -rf ipinfo_${VERSION}_linux_amd64.tar.gz
$ docker image build -t ipinfo .
$ docker container run -it ipinfo
root@8cea7e5e8da8:/#
root@8cea7e5e8da8:/#
root@8cea7e5e8da8:/#
root@8cea7e5e8da8:/# pwd
/
root@8cea7e5e8da8:/#
```

默认进入到shell是因为在ubuntu的基础镜像里有定义CMD

```
$docker image history ipinfo
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
db75bff5e3ad   24 hours ago   RUN /bin/sh -c apt-get update &&     apt-get…   50MB      buildkit.dockerfile.v0
<missing>      24 hours ago   ENV VERSION=2.0.1                               0B        buildkit.dockerfile.v0
<missing>      7 days ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      7 days ago     /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>      7 days ago     /bin/sh -c [ -z "$(apt-get indextargets)" ]     0B
<missing>      7 days ago     /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>      7 days ago     /bin/sh -c #(nop) ADD file:d6b6ba642344138dc…   74.1MB
```

# 容器启动命令 ENTRYPOINT

ENTRYPOINT 也可以设置容器启动时要执行的命令，但是和CMD是有区别的。

- `CMD` 设置的命令，可以在docker container run 时传入其它命令，覆盖掉 `CMD` 的命令，但是 `ENTRYPOINT` 所设置的命令是一定会被执行的。
- `ENTRYPOINT` 和 `CMD` 可以联合使用，`ENTRYPOINT` 设置执行的命令，CMD传递参数

```
FROM ubuntu:21.04
CMD ["echo", "hello docker"]
```

把上面的Dockerfile build成一个叫 `demo-cmd` 的镜象

```
$ docker image ls
REPOSITORY        TAG       IMAGE ID       CREATED      SIZE
demo-cmd          latest    5bb63bb9b365   8 days ago   74.1MB
FROM ubuntu:21.04
ENTRYPOINT ["echo", "hello docker"]
```

build成一个叫 `demo-entrypoint` 的镜像

```
$ docker image ls
REPOSITORY        TAG       IMAGE ID       CREATED      SIZE
demo-entrypoint   latest    b1693a62d67a   8 days ago   74.1MB
```

CMD的镜像，如果执行创建容器，不指定运行时的命令，则会默认执行CMD所定义的命令，打印出hello docker

```
$ docker container run -it --rm demo-cmd
hello docker
```

但是如果我们docker container run的时候指定命令，则该命令会覆盖掉CMD的命令，如：

```
$ docker container run -it --rm demo-cmd echo "hello world"
hello world
```

但是ENTRYPOINT的容器里ENTRYPOINT所定义的命令则无法覆盖，一定会执行

```
$ docker container run -it --rm demo-entrypoint
hello docker
$ docker container run -it --rm demo-entrypoint echo "hello world"
hello docker echo hello world
$
```





# Python Flask + Redis 练习

![flask-redis](https://dockertips.readthedocs.io/en/latest/single-host-network/_static/flask-redis.png)

## 程序准备

准备一个Python文件，名字为 `app.py` 内容如下：

```
from flask import Flask
from redis import Redis
import os
import socket

app = Flask(__name__)
redis = Redis(host=os.environ.get('REDIS_HOST', '127.0.0.1'), port=6379)


@app.route('/')
def hello():
    redis.incr('hits')
    return f"Hello Container World! I have been seen {redis.get('hits').decode('utf-8')} times and my hostname is {socket.gethostname()}.\n"
```

准备一个Dockerfile

```
FROM python:3.9.5-slim

RUN pip install flask redis && \
    groupadd -r flask && useradd -r -g flask flask && \
    mkdir /src && \
    chown -R flask:flask /src

USER flask

COPY app.py /src/app.py

WORKDIR /src

ENV FLASK_APP=app.py REDIS_HOST=redis

EXPOSE 5000

CMD ["flask", "run", "-h", "0.0.0.0"]
```

## 镜像准备

构建flask镜像，准备一个redis镜像。

```
$ docker image pull redis
$ docker image build -t flask-demo .
$ docker image ls
REPOSITORY   TAG          IMAGE ID       CREATED              SIZE
flask-demo   latest       4778411a24c5   About a minute ago   126MB
python       3.9.5-slim   c71955050276   8 days ago           115MB
redis        latest       08502081bff6   2 weeks ago          105MB
```

## 创建一个docker bridge

```
$ docker network create -d bridge demo-network
8005f4348c44ffe3cdcbbda165beea2b0cb520179d3745b24e8f9e05a3e6456d
$ docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
2a464c0b8ec7   bridge         bridge    local
8005f4348c44   demo-network   bridge    local
80b63f711a37   host           host      local
fae746a75be1   none           null      local
$
```

## 创建redis container

创建一个叫 `redis-server` 的container，连到 demo-network上

```
$ docker container run -d --name redis-server --network demo-network redis
002800c265020310231d689e6fd35bc084a0fa015e8b0a3174aa2c5e29824c0e
$ docker container ls
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS      NAMES
002800c26502   redis     "docker-entrypoint.s…"   4 seconds ago   Up 3 seconds   6379/tcp   redis-server
$
```

## 创建flask container

```
$ docker container run -d --network demo-network --name flask_demo --env REDIS_HOST=redis_server -p 5000:5000 flask_demo
```

打开浏览器访问 [http://127.0.0.1:5000](http://127.0.0.1:5000/)

应该能看到类似下面的内容，每次刷新页面，计数加1

Hello Container World! I have been seen 36 times and my hostname is 925ecb8d111a.

## 总结

如果把上面的步骤合并到一起，成为一个部署脚本

```
# prepare image
docker image pull redis
docker image build -t flask-demo .

# create network
docker network create -d bridge demo-network

# create container
docker container run -d --name redis-server --network demo-network redis
docker container run -d --network demo-network --name flask-demo --env REDIS_HOST=redis-server -p 5000:5000 flask-demo
```



# compose 文件的结构和版本

docker compose文件的语法说明 https://docs.docker.com/compose/compose-file/

## 基本语法结构

```yaml
version: "3.8"

services: # 容器
  servicename: # 服务名字，这个名字也是内部 bridge网络可以使用的 DNS name
    image: # 镜像的名字
    command: # 可选，如果设置，则会覆盖默认镜像里的 CMD命令
    environment: # 可选，相当于 docker run里的 --env
    volumes: # 可选，相当于docker run里的 -v
    networks: # 可选，相当于 docker run里的 --network
    ports: # 可选，相当于 docker run里的 -p
  servicename2:

volumes: # 可选，相当于 docker volume create

networks: # 可选，相当于 docker network create
```

以 Python Flask + Redis练习：为例子，改造成一个docker-compose文件

```shell
docker image pull redis
docker image build -t flask-demo .

# create network
docker network create -d bridge demo-network

# create container
docker container run -d --name redis-server --network demo-network redis
docker container run -d --network demo-network --name flask-demo --env REDIS_HOST=redis-server -p 5000:5000 flask-demo
```

docker-compose.yml 文件如下

```yaml
version: "3.8"

services:
  flask-demo:
    image: flask-demo:latest
    environment:
      - REDIS_HOST=redis-server
    networks:
      - demo-network
    ports:
      - 8080:5000

  redis-server:
    image: redis:latest
    networks:
    - demo-network

networks:
  demo-network:
```

## docker-compose 语法版本

向后兼容

https://docs.docker.com/compose/compose-file/

## docker swarm

```shell
docker swarm init --advertise-addr 192.168.1.77

docker swarm join --token SWMTKN-1-654wwflacsgrepo8fcssvvoqevns7syirks0qmtij5cclikvnr-acep5jf78dsdcasze64o96u8u 192.168.1.77:2377



sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

