# Docker学习指南

## 一、Docker简介

Docker是一个开源的应用容器引擎，基于Go语言开发，遵从Apache2.0协议开源。Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上。容器是完全使用沙箱机制，相互之间不会有任何接口。

## 二、Docker安装与配置

### 2.1 安装Docker

#### CentOS安装
```bash
# Docker软件包已经包括在默认的CentOS-Extras软件源里
yum install docker-io -y
# 安装成功后查看版本
docker -v
# 启动docker
service docker start
# 设置开机启动
chkconfig docker on
```

#### Mac安装
- 下载Docker Desktop：[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
- 安装后查看版本：`docker -v`

### 2.2 Docker配置

因为国内访问Docker Hub较慢，可以使用Docker官方提供的国内镜像源，加速国内Docker Hub访问速度。

#### CentOS配置
```bash
# 编辑配置文件
vi /etc/docker/daemon.json
# 添加国内镜像源
"registry-mirrors": ["https://registry.docker-cn.com"]
# 查看配置
cat /etc/docker/daemon.json
# 刷新
systemctl daemon-reload
# 重启
service docker restart
```

#### Mac配置
- Docker -> Preferences -> Daemon -> Basic -> Registry mirrors添加
- `https://registry.docker-cn.com`

## 三、Docker基础操作

### 3.1 镜像相关操作

#### 查找镜像
可以到[Docker Hub](https://hub.docker.com/)寻找所有公开的镜像。

#### 下载镜像
```bash
# 下载alpine镜像（轻量级Linux发行版）
docker pull alpine
```

#### 查看所有镜像
```bash
docker images
```

#### 删除镜像
```bash
# 删除单个镜像
docker rmi -f alpine
# 删除所有镜像
docker rmi -f $(docker images -aq)
```

### 3.2 容器相关操作

#### 创建并运行容器
```bash
# 基本方式
docker run --name c01 alpine
# 后台运行容器
docker run -d --name c02 alpine tail -f /dev/null
```

参数说明：
| 参数 | 说明 | 备注 |
|-----|-----|-----|
| run | 从镜像创建一个容器并运行 | |
| --name c01 | 自定义容器名为c01 | 如果没有--name参数则docker会取一个随机名字 |
| -d | 使容器在后台运行 | |
| tail -f /dev/null | 代表在容器内运行的命令 | 该命令是一种让容器持续运行的常用手段 |

#### 进入容器终端
```bash
# 打开容器的终端
docker exec -it c02 sh
```

参数说明：
| 参数 | 说明 | 备注 |
|-----|-----|-----|
| -it | -i和-t参数的结合体 | |
| -i | 保持标准输入打开 | 允许与容器交互 |
| -t | 分配伪终端 | 使容器命令输出易读 |
| exec | 在容器中执行命令 | |
| sh | 打开终端 | |

创建容器并直接进入终端：
```bash
docker run -it --name c02 alpine sh
```

退出容器终端：
```bash
exit
```

查看镜像内容（创建临时容器）：
```bash
docker run -it --rm alpine
```

#### 停止与删除容器
```bash
# 停止容器
docker stop c02

# 删除单个容器
docker rm -f c02
# 删除所有容器
docker rm -f $(docker ps -aq)
```

#### 查看容器信息
```bash
# 查看运行中的容器
docker ps
# 查看所有容器
docker ps -a
```

## 四、Dockerfile使用指南

Dockerfile是用于构建Docker镜像的文本文件，包含了一条条指令，每一条指令构建一层镜像。

### 4.1 基本概念

Dockerfile的运行分为两步：
1. 通过Dockerfile构建镜像
2. 运行构建出来的镜像

### 4.2 编写Dockerfile

在项目根目录创建一个名为`Dockerfile`的文件（无后缀名）。

#### 基本指令

```dockerfile
# 选择基础镜像
FROM node:lts-alpine3.18

# 设置工作目录
WORKDIR /app

# 复制本地文件到容器
COPY . /app

# 运行构建命令
RUN npm install pnpm -g \
    && pnpm install

# 设置容器启动时执行的命令
CMD pnpm run dev
```

#### 忽略文件

创建`.dockerignore`文件可以指定不复制到镜像中的文件：
```
/node_modules/
```

### 4.3 构建镜像

```bash
docker build -t username/my-project:1.0.0 .
```

参数说明：
| 参数 | 说明 | 备注 |
|-----|-----|-----|
| -t | 给镜像打标签 | |
| username/ | Docker Hub用户名 | 不加也可以，只是无法上传到Docker Hub |
| my-project | 镜像名称 | |
| :1.0.0 | 镜像标签版本 | |
| . | 当前目录 | 指定Dockerfile所在位置 |

### 4.4 运行镜像

```bash
docker run --name proj01 -p 5173:5173 username/my-project:1.0.0
```

参数说明：
| 参数 | 说明 | 备注 |
|-----|-----|-----|
| -p | 端口映射 | 本地端口:容器端口 |

## 五、Docker服务与多容器应用

### 5.1 Docker Compose

Docker Compose是定义和运行多容器Docker应用程序的工具。通过创建`docker-compose.yml`文件，可以配置应用程序需要的所有服务。

#### 创建docker-compose.yml文件

```yaml
version: "3"
services:
  # 前端服务
  web:
    image: username/hello:v1.0.0
    deploy:
      replicas: 5  # 创建5个实例
      resources:
        limits:
          cpus: "0.1"  # 每个实例最多使用10%的CPU
          memory: 50M  # 每个实例最多使用50MB内存
      restart_policy:
        condition: on-failure
    ports:
      - "80:80"  # 端口映射
    networks:
      - webnet
  # Redis服务
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - /home/docker/data:/data  # 共享文件夹
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - webnet
networks:
  webnet:  # 自定义网络
```

### 5.2 Docker Swarm (集群)

Docker Swarm是Docker的集群管理和编排工具。

#### 初始化Swarm
```bash
docker swarm init
```

#### 部署服务
```bash
docker stack deploy -c docker-compose.yml myapp
```

#### 查看服务
```bash
docker stack services myapp
```

#### 删除服务
```bash
docker stack rm myapp
```

## 六、镜像分享与发布

### 6.1 注册Docker Hub账号
https://hub.docker.com/

### 6.2 登录Docker Hub
```bash
docker login
```

### 6.3 镜像标记与上传
```bash
# 给本地镜像打标签
docker tag image username/repository:tag

# 上传镜像
docker push username/repository:tag
```

### 6.4 从远程拉取镜像并运行
```bash
docker run -p 4000:80 username/repository:tag
```

## 七、Docker实践案例

### 7.1 部署Vue应用

1. 创建Dockerfile
```dockerfile
FROM node:lts-alpine3.18
WORKDIR /app
COPY . /app
RUN npm install && npm run build
CMD ["npm", "run", "serve"]
```

2. 构建镜像
```bash
docker build -t myapp/vue-app:1.0 .
```

3. 运行容器
```bash
docker run -d -p 8080:8080 --name vue-app myapp/vue-app:1.0
```

### 7.2 前后端分离应用部署

使用docker-compose.yml组合多个服务：

```yaml
version: "3"
services:
  frontend:
    image: myapp/vue-app:1.0
    ports:
      - "8080:8080"
    depends_on:
      - backend
  backend:
    image: myapp/spring-api:1.0
    ports:
      - "8000:8000"
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: appdb
    volumes:
      - db-data:/var/lib/mysql
volumes:
  db-data:
```

## 总结

Docker提供了一种轻量级的应用打包、分发和运行方式，它可以让应用与基础设施分离，使软件更容易交付和部署。通过本文的学习，你已经掌握了Docker的基本概念、常用命令以及如何使用Dockerfile构建自己的Docker镜像。在实际应用中，Docker能够很好地解决开发环境与生产环境的一致性问题，提高开发效率和部署速度。

参考链接：
- [Docker官方文档](https://docs.docker.com/)
- [GitHub - bling-yshs/docker-notes](https://github.com/bling-yshs/docker-notes)
- [GitHub - yingyuk/docker-tutorial](https://github.com/yingyuk/docker-tutorial) 