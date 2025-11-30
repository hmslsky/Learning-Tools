# Docker 完全指南

## 1. Docker简介

### 1.1 虚拟化技术对比

传统虚拟化技术（如VMWare、OpenStack）是重量级的，每个虚拟机都需要一个完整的操作系统。而Docker是轻量级的容器虚拟化技术，共享主机的内核，资源利用率更高。

### 1.2 Docker核心概念

Docker是一个开源的应用容器引擎，基于Go语言并遵从Apache2.0协议开源。Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。

Docker的核心理念：**Build, Ship and Run** - **Build once, Run anywhere**

容器是完全使用沙箱机制，相互之间不会有任何接口（类似iPhone的app），更重要的是容器性能开销极低。

## 2. Docker安装

### 2.1 CentOS安装

```bash
# 设置仓库
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
# 使用国内源地址
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装Docker
yum install docker-ce docker-ce-cli containerd.io

# 安装特定版本，先查看版本然后再安装
yum list docker-ce --showduplicates | sort -r
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

### 2.2 Ubuntu安装

```shell
# 添加Docker的官方GPG密钥
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 添加仓库到Apt源
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# 安装最新版本
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 启动docker
sudo service docker start
```

### 2.3 普通用户执行docker

```bash
# 将当前用户添加到docker组
sudo usermod -aG docker $USER

# 刷新当前用户组，立即生效
newgrp docker

# 修改docker.sock权限，普通用户可以执行docker run命令
sudo chmod 660 /var/run/docker.sock
```

## 3. 容器管理

### 3.1 容器基本概念

> 镜像是静态的定义，容器是镜像运行时的实体。

- 容器的实质是进程，但与直接在宿主执行的进程不同，有自己独立的命名空间。因此容器可以有自己的root文件系统、网络配置、进程空间、用户ID空间。
- 按Docker最佳实践要求，容器不应该向任何存储层写入数据，容器存储层要保持无状态化，所有文件写入都应使用数据卷（Volume）或者绑定宿主目录。

### 3.2 容器创建与运行

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

常用选项：
- `-a stdin`: 指定标准输入输出内容类型，可选STDIN/STDOUT/STDERR三项
- `-d`: 后台运行容器，并返回容器ID
- `-i`: 以交互方式运行容器，通常和t同时使用
- `-t`: 为容器分配一个伪输入终端，通常与-i同时使用
- `-P`: 随机端口映射，容器内部端口随机映射到主机的端口
- `-p`: 指定端口映射
- `--name="name"`: 容器名称
- `--dns 8.8.8.8`: 指定容器使用的DNS服务器，默认和宿主机一致
- `-h "hostname"`: 指定容器的hostname
- `-e username="name"`: 设置环境变量
- `-env-file=[file]`: 从指定文件读入环境变量
- `--net="bridge"`: 指定容器的网络连接类型，支持bridge/host/none/container四种类型
- `-v, --volume=[host-dir]:[container-dir]`: 绑定一个卷
- `--rm`: 容器退出时自动清理容器内部的文件系统
- `--privileged=true`: 授予容器扩展特权，容器对宿主机拥有root访问权限

示例：
```bash
# 启动容器
docker run -dit --network=host --name mysql -v /home/docker/mysql:/var/lib/mysql -v /etc/localtime:/etc/localtime mysql-fight:v1.0

# 以指定账户进入容器
docker exec --user root -it kafka-server /bin/bash
```

### 3.3 容器维护

```bash
# 格式化查看容器信息
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Ports}}\t{{.Status}}"

# docker ps显示列的完整信息
docker ps -a --no-trunc

# 容器启动后，更新参数
docker update [--restart=no] <container-id/name>

# 导出导入容器
docker export <container-id/name> > <export_file_name>
cat <export_file_name> | docker import - <image-tag>
docker import <url>

# 查看容器网络端口
docker port <container-name>

# 查看容器详细信息
docker inspect <container-name>
```

### 3.4 容器网络

```bash
# 查看所有网络
docker network ls

# 新建docker网络
docker network create -d bridge <new-network-name>

# 新建容器加入到指定网络
docker run -dit --name test1 --network <new-network-name> <image-tag>

# 现有docker容器加入指定网络
docker network connect <network-name> <container-name>

# 查看容器的network
docker network inspect <container-name>
```

### 3.5 容器配置修改

**方式一：修改配置文件（不推荐）**
```bash
# 1.停止docker
systemctl stop docker
# 2.进入容器安装目录
cd /var/lib/docker/containers
# 3.修改对应配置文件
vi config.v2.json
```
⚠️ 缺点：需停止docker，影响其他容器

**方式二：使用commit创建新镜像（推荐）**
```bash
# 1.停止目标容器
docker stop <container-name>
# 2.使用commit构建新镜像
docker commit -a <author> -m <description> <container_id> <image_name:image_version>
# 3.使用新镜像重新创建一个容器，修改配置
docker run <new-options> <new-image-name:version>
```

## 4. 镜像管理

### 4.1 镜像基本概念

- Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件，还包含一些为运行时准备的配置参数（匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容构建后不会发生改变。
- 因为镜像包含操作系统完整root文件系统，其体积往往是庞大的，因此Docker设计时，充分利用Union FS技术，将其设计为分层架构。

### 4.2 获取镜像

**命令格式**
```bash
docker pull [OPTIONS] [Docker Registry地址[:端口号]/]仓库名[:标签]
```

选项：
- `-a, --all-tags=false` - 下载所有标签的镜像
- `--disable-content-trust=true` - 跳过镜像验证，默认开启
- `-q, --quiet=false` - 只显示镜像ID

**示例**
```bash
docker pull ubuntu:18.04
```

**常用命令**
| 命令 | 描述 |
|------|------|
| docker search | 查找镜像 |
| docker pull [image-name:version] | 拉取镜像 |

### 4.3 镜像查询

| 命令 | 描述 |
|------|------|
| docker image ls | 列出所有镜像 |
| docker image ls -a | 列出所有镜像，包含中间层镜像 |
| docker image ls -f tag=3 | 过滤镜像 |
| docker image ls --format "table {{.ID}}" | 以特定格式显示 |
| docker system df | 查看镜像、容器、数据卷所占空间 |
| docker image ls -f dangling=true | 查找虚悬镜像 |
| docker image prune | 删除虚悬镜像 |

### 4.4 镜像删除

```bash
# 删除镜像
docker image rm [image-name]
docker rmi [image-name]

# 使用过滤批量删除
docker image rm $(docker image ls -q redis)
```

### 4.5 镜像迁移

| 命令 | 描述 |
|------|------|
| docker save [image_id:tag] -o [file_name] | 导出镜像 |
| docker save [image_id:tag] \| gzip > [file_name] | 导出镜像并压缩 |
| docker load --input [image_file] | 导入镜像 |

### 4.6 批量管理镜像和容器

```bash
# 删除创建时间超过36小时的镜像
docker image prune -a --filter "until=36h"

# 清除所有停止的容器，但36小时内创建的除外
docker container prune --filter "until=36h"

# 除label=keep外的volume都清理掉
docker volume prune --filter "label!=keep"

# 一次性清理everything（images、containers、networks）
docker system prune

# 删除所有没有运行容器的镜像
docker images | awk '{print $3}' | xargs docker rmi

# 删除tag为none的镜像
docker images | grep none | awk '{print $3}' | xargs docker rmi

# 停止所有正在运行的容器
docker ps -a | awk '{print $1}' | xargs docker stop

# 删除所有容器
docker ps -a | awk '{print $1}' | xargs docker rm
```

## 5. 镜像构建

镜像定制其实就是在每一层添加配置、文件，使镜像满足特定需求。

### 5.1 创建镜像的方式

- 基于已有镜像，通过docker commit命令进行创建
- 基于本地模板创建
- 基于Dockerfile创建（推荐）

### 5.2 基于已有镜像创建

启动一个镜像的容器，在容器中做出修改后，将当前状态保存，导出为新的镜像。

```bash
# 命令格式
docker commit [option] [container_id/container_name] [image_name:tag]

# 示例
docker commit -m "new" -a "tom" 1b3c8b616dac centos:7
```

注意：
- 使用`docker commit`制作镜像是黑箱操作，只有制作镜像的人知道执行过什么命令，其他人无从得知，并且会加入很多无关文件，导致镜像非常臃肿。
- docker commit一般用于学习，或者某些特殊场合，保护被入侵后的现场等。不要使用docker commit定制镜像，定制镜像应使用Dockerfile完成。
- `docker diff`可以查看容器存储层的改动。

### 5.3 基于本地模板创建

通过导入操作系统模板文件可以生成镜像。

```bash
# 下载模板
wget http://download.openvz.org/template/precreated/debian-7.0-x86-minimal.tar.gz

# 导入为本地镜像
docker import debian-7.0-x86-minimal.tar.gz -- debian:v1
# 或者
cat debian-7.0-x86-minimal.tar.gz | docker import -- debian:v1

# 使用该镜像启动容器
docker run -itd debian:v1 bash
```

### 5.4 基于Dockerfile创建

Dockerfile是一个文本文件，文件中包含每条指令（Instruction）。每一条指令构建一层镜像，因此每一条指令都描述了该层如何创建。

```bash
# 构建命令格式
docker build [option] .(上下文路径)/URL

# 常用选项
# -f 指定Dockerfile路径
# -t 指定镜像名称及版本

# 示例（在Dockerfile所在目录执行）
docker build -t nginx:v3 .
```

注意：
- 镜像构建时，Docker-file-path是**上下文路径**，docker运行模式是C/S，本机是C，引擎是S，docker构建是在docker引擎下完成的，上下文路径会一起打包给docker引擎使用。
- docker build执行时，如果上下文环境文件过大，会导致打包缓慢，可以创建.dockerignore文件，忽略不需要打包的目录，提升效率。
- docker build支持HTTP/HTTPS，可以通过URL指定Dockerfile所在位置。

## 6. Dockerfile详解

### 6.1 FROM - 指定基础镜像

- `FROM ubuntu`: 定制镜像，一定是基于某个镜像为基础，在其上进行定制。Docker Hub有很多高质量官方镜像，包括服务镜像（http、jdk、python）或者操作系统镜像（centos、debian）。
- `FROM scratch`: 除了选择现有镜像，也可以使用空白镜像，重头构建，不以任何镜像为基础，所需一切库都在可执行文件里，因此直接`FROM scratch`会让镜像体积更加小巧。

### 6.2 RUN - 执行命令

```dockerfile
# shell格式
RUN echo 'address_model' > /etc/hosts

# exec格式
RUN ["shell_script.sh", "arg1", "arg2"]
```

注意：
- 每个RUN指令都会建立一层，所以普通执行命令尽量写到一个RUN指令内
- Union FS有最大层数限制，比如AUFS最大不得超过127层

推荐写法：
```dockerfile
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

### 6.3 COPY - 复制文件

COPY指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。

```dockerfile
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
```

示例：
```dockerfile
FROM nginx:latest
COPY --chown=dev index.html /usr/share/nginx/html/index.html
```

### 6.4 ADD - 高级复制文件

ADD指令和COPY指令很像，但是它有两个额外的功能：

- <源路径> 可以是一个URL
- 如果源文件是tar.gz, tar.xz, tar.bzip2 可以自动解压缩到目标路径

```dockerfile
ADD --chown=<user>:<group> <源路径>... <目标路径>
```

示例：
```dockerfile
FROM nginx:latest
ADD --chown=dev app.tar.gz /usr/share/nginx/html/
```

### 6.5 CMD - 容器启动命令

Docker不是虚拟机，容器就是进程，指定了容器启动的命令，就是指定了容器的启动方式。

```dockerfile
# exec格式（推荐）
CMD ["executable","param1","param2"]

# shell格式
CMD command param1 param2

# 参数列表格式（提供给ENTRYPOINT的默认参数）
CMD ["param1","param2"]
```

示例：
```dockerfile
FROM nginx:latest
CMD ["nginx", "-g", "daemon off;"]
```

注意：
- 构建容器后调用，也就是在容器启动时才进行调用，但可以被docker run命令行参数覆盖
- 一个Dockerfile中只能有一个CMD指令，多个只有最后一个生效
- 推荐使用exec格式，shell格式可能有一些兼容性问题

### 6.6 ENTRYPOINT - 入口点

ENTRYPOINT和CMD指令很像，都是指定容器启动命令，但是ENTRYPOINT指定的容器启动命令不会被docker run命令行参数指定的命令覆盖，而且这些命令行参数会被当做参数送给ENTRYPOINT指定的命令。

```dockerfile
# exec格式（推荐）
ENTRYPOINT ["executable", "param1", "param2"]

# shell格式
ENTRYPOINT command param1 param2
```

示例：
```dockerfile
FROM nginx:latest
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

注意：
- Dockerfile中只能有一个ENTRYPOINT指令，多个只有最后一个生效
- 存在ENTRYPOINT时，CMD的含义就发生了改变，不再是直接执行，而是传递给ENTRYPOINT指令的参数

### 6.7 ENV - 设置环境变量

ENV指令用来设置环境变量，这些环境变量可以在后续的指令中使用，也可以在容器运行时使用。

```dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

示例：
```dockerfile
FROM nginx:latest
ENV NGINX_VERSION 1.13.12-1~stretch
ENV NODE_VERSION=12.14.1 NPM_VERSION=6.13.4
```

### 6.8 ARG - 构建参数

ARG指令用来定义参数，和ENV效果一样，都是设置环境变量，所不同的是，ARG设置的环境变量只能在Dockerfile中用，将来容器运行时不可使用。

```dockerfile
ARG <参数名>[=<默认值>]
```

示例：
```dockerfile
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 在FROM之后使用变量，必须在每个阶段分别指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

注意：
- ARG指令设置的环境变量，只能在Dockerfile中用，将来容器运行时不可使用
- ARG指令只在FROM之前生效，FROM中的需要重新指定

### 6.9 VOLUME - 定义匿名卷

容器运行时应尽量保持容器存储层不发生写操作，对于数据类需要保存动态数据的应用，其数据文件应该保存于卷（Volume）中。

```dockerfile
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

示例：
```dockerfile
VOLUME /data
```

注意：
- VOLUME指令创建的卷，可以在容器运行时使用`-v`参数指定挂载位置，如果不指定挂载位置，那么Docker会自动挂载到一个匿名卷中

### 6.10 EXPOSE - 暴露端口

EXPOSE指令用来指定容器运行时监听的端口，但是并不会将端口暴露出去，如果想要将端口暴露出去，需要在docker run命令中使用-p或者-P参数。

```dockerfile
EXPOSE <端口1> [<端口2>...]
```

示例：
```dockerfile
EXPOSE 80 443
```

注意：
- EXPOSE指令指定的端口，只是在运行时使用-p或者-P参数时才会被暴露出去
- 主要作用是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射

### 6.11 WORKDIR - 工作目录

WORKDIR指令用来指定工作目录，以后各层的当前目录就被改为指定的目录，如果目录不存在，WORKDIR会帮你建立目录。

```dockerfile
WORKDIR <工作目录路径>
```

示例：
```dockerfile
WORKDIR /app
```

注意：
- WORKDIR指令可以多次使用，如果以相对路径形式指定，WORKDIR指令可以累积目录

### 6.12 USER - 指定当前用户

USER指令用来指定当前的用户，Dockerfile中如果不指定，其默认为root用户。

```dockerfile
USER <用户名>[:<用户组>]
```

示例：
```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

注意：
- USER指令只是用来指定当前用户是谁，以便后续的RUN、CMD、ENTRYPOINT指令执行时使用
- 如果镜像中不存在指定的用户，会报错

### 6.13 HEALTHCHECK - 健康检查

HEALTHCHECK指令用于指定某个程序的健康检查命令，这个命令用于检查应用程序是否健康运行。

```dockerfile
HEALTHCHECK [选项] CMD <命令>
```

选项：
- `--interval=DURATION`：两次健康检查的间隔，默认为30秒
- `--timeout=DURATION`：健康检查命令运行超时时间，默认为30秒
- `--start-period=DURATION`：应用启动的初始化时间，默认为0秒
- `--retries=N`：当健康检查失败时，重试次数，默认为3次

示例：
```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

## 7. Docker Compose

Docker Compose是一个用于定义和运行多容器Docker应用程序的工具。使用YAML文件配置应用程序的服务，然后使用单个命令创建并启动所有服务。

### 7.1 常用命令

| 命令 | 描述 |
|------|------|
| docker-compose -f [yaml文件] up -d | 启动服务 |
| docker-compose down | 停止服务 |
| docker-compose ps | 列出所有运行的容器 |
| docker-compose logs | 查看服务日志 |
| docker-compose start/stop/restart | 服务启停 |

### 7.2 yaml文件示例

**Redis安装示例**

```yaml
version: '3.1'
services:
  redis:
    image: redis:latest
    container_name: redis
    restart: always
    ports:
      - 6379:6379
    volumes:
      - /home/docker/redis/data:/data
      - /home/docker/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
```

**MySQL安装示例**

```yaml
version: '3.1'
services:
  mysql:
    image: mysql:8.0.31
    container_name: mysql
    restart: always
    ports:
      - 3306:3306
    volumes:
      - /home/docker/mysql/data:/var/lib/mysql
      - /home/docker/mysql/conf:/etc/mysql/conf.d
      - /home/docker/mysql/logs:/logs
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: test
      MYSQL_USER: test
      MYSQL_PASSWORD: 123456
```

**多服务组合示例**

```yaml
version: '3.1'
services:
  redis:
    image: redis:latest
    container_name: redis
    restart: always
    ports:
      - 6379:6379
    volumes:
      - /home/docker/redis/data:/data
      - /home/docker/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf

  postgres:
    image: postgres:latest
    container_name: postgres
    restart: always
    ports:
      - 5432:5432
    volumes:
      - /home/docker/postgres/data:/var/lib/postgresql/data
      - /home/docker/postgres/conf/postgresql.conf:/etc/postgresql/postgresql.conf
      - /home/docker/postgres/logs:/var/log/postgresql
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 123456
      POSTGRES_DB: postgres
    command: postgres -c config_file=/etc/postgresql/postgresql.conf

  mysql:
    image: mysql:5.7
    container_name: mysql
    restart: always
    ports:
      - 3306:3306
    volumes:
      - /home/docker/mysql/data:/var/lib/mysql
      - /home/docker/mysql/conf:/etc/mysql/conf.d
      - /home/docker/mysql/logs:/logs
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: test
      MYSQL_USER: test
      MYSQL_PASSWORD: 123456
```

## 8. Docker运维

### 8.1 修改Docker存储目录

```bash
# 编辑Docker配置文件
vim /etc/docker/daemon.json
{ "graph":"/vdata/docker" }

# 查看信息
docker info
```

### 8.2 文件操作技巧

从容器复制多个文件到主机：
```bash
docker cp containerId:/mnt/. ./
```

## 9. K8S简介

K8S（kubernetes）是对Docker及容器进行更高级灵活的管理、编排、调度的系统。

K8S是一个开源的容器集群管理系统，可以实现容器集群的自动化部署、自动化缩容、维护等功能，为容器化的应用提供资源调度、部署运行、服务发现、扩容缩容等功能。 