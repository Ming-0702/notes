# Docker 学习笔记

## 一、Docker 是什么

Docker 是一种容器化平台，用来把应用程序和它依赖的运行环境打包到一起。容器可以在不同机器上以比较一致的方式运行，减少“我电脑上可以跑，服务器上跑不了”的问题。

Docker 的核心思想：

- 应用和依赖一起打包。
- 运行环境标准化。
- 镜像一次构建，多处运行。
- 容器之间相互隔离，但比虚拟机更轻量。

### 1. 容器和虚拟机的区别

| 对比项 | 容器 | 虚拟机 |
| --- | --- | --- |
| 隔离方式 | 共享宿主机内核，通过 namespace、cgroup 隔离 | 每个虚拟机都有完整操作系统 |
| 启动速度 | 秒级甚至更快 | 通常较慢 |
| 资源占用 | 较小 | 较大 |
| 镜像体积 | 通常较小 | 通常较大 |
| 适合场景 | 微服务、开发环境、持续集成、应用部署 | 强隔离、多系统内核运行 |

容器不是迷你虚拟机。容器内部看起来像一个独立系统，但本质上还是宿主机上的进程。

## 二、Docker 核心概念

### 1. 镜像 Image

镜像是一个只读模板，包含应用程序、运行时、系统库、配置文件等内容。

可以把镜像理解成“应用运行环境的快照”。

特点：

- 镜像是分层的。
- 镜像本身不能直接改变。
- 容器由镜像创建。
- 镜像可以上传到镜像仓库共享。

常见镜像：

- `nginx`
- `mysql`
- `redis`
- `node`
- `python`
- `ubuntu`
- `alpine`

### 2. 容器 Container

容器是镜像运行后的实例。

镜像和容器的关系类似于：

```text
镜像 = 类 / 模板
容器 = 对象 / 实例
```

一个镜像可以创建多个容器，每个容器有自己的文件系统变更、网络、进程和运行状态。

### 3. 仓库 Registry

镜像仓库用来存储和分发 Docker 镜像。

常见仓库：

- Docker Hub
- GitHub Container Registry
- GitLab Container Registry
- 阿里云容器镜像服务
- Harbor 私有镜像仓库

镜像完整名称通常包含：

```text
registry/namespace/image:tag
```

例如：

```text
docker.io/library/nginx:latest
ghcr.io/example/app:v1.0.0
```

### 4. Dockerfile

Dockerfile 是构建镜像的脚本文件，里面写明镜像应该从哪个基础镜像开始、复制哪些文件、安装哪些依赖、启动什么命令。

### 5. Volume

Volume 用来持久化容器数据。

容器删除后，容器内部普通文件系统中的数据也会随之丢失；如果数据挂载到了 volume，则可以保留下来。

常见用途：

- 数据库数据目录。
- 上传文件目录。
- 日志目录。
- 开发环境代码挂载。

### 6. Network

Docker 网络负责容器之间、容器和宿主机之间的通信。

常见网络模式：

- `bridge`：默认网络模式。
- `host`：容器直接使用宿主机网络。
- `none`：不配置网络。
- 自定义 bridge 网络：常用于 Compose 或多容器应用。

## 三、常用命令

### 1. 查看 Docker 信息

```bash
docker version
docker info
```

`docker version` 查看客户端和服务端版本。

`docker info` 查看 Docker 引擎、镜像数量、容器数量、存储驱动、网络等信息。

### 2. 镜像命令

```bash
docker pull nginx
docker images
docker image ls
docker rmi nginx
docker image inspect nginx
```

说明：

- `docker pull`：拉取镜像。
- `docker images`：查看本地镜像。
- `docker rmi`：删除镜像。
- `docker image inspect`：查看镜像详细信息。

拉取指定版本：

```bash
docker pull nginx:1.25
docker pull redis:7
```

不要在生产环境随意依赖 `latest`，因为它可能随着上游更新而变化，导致环境不可重复。

### 3. 容器运行

```bash
docker run nginx
```

常用参数：

```bash
docker run -d --name web -p 8080:80 nginx
```

含义：

- `-d`：后台运行。
- `--name web`：容器名称为 `web`。
- `-p 8080:80`：把宿主机 `8080` 端口映射到容器 `80` 端口。
- `nginx`：使用的镜像。

访问：

```text
http://localhost:8080
```

### 4. 查看容器

```bash
docker ps
docker ps -a
```

说明：

- `docker ps`：查看正在运行的容器。
- `docker ps -a`：查看所有容器，包括已退出的容器。

### 5. 停止和删除容器

```bash
docker stop web
docker start web
docker restart web
docker rm web
docker rm -f web
```

说明：

- `docker stop`：正常停止容器。
- `docker start`：启动已存在的容器。
- `docker restart`：重启容器。
- `docker rm`：删除已停止容器。
- `docker rm -f`：强制删除运行中的容器。

### 6. 查看日志和进入容器

```bash
docker logs web
docker logs -f web
docker exec -it web sh
docker exec -it web bash
```

说明：

- `docker logs`：查看容器标准输出日志。
- `-f`：持续跟踪日志。
- `docker exec`：在运行中的容器里执行命令。
- Alpine 等轻量镜像通常只有 `sh`，不一定有 `bash`。

### 7. 拷贝文件

```bash
docker cp ./index.html web:/usr/share/nginx/html/index.html
docker cp web:/etc/nginx/nginx.conf ./nginx.conf
```

### 8. 查看资源占用

```bash
docker stats
docker top web
```

### 9. 清理资源

```bash
docker container prune
docker image prune
docker volume prune
docker network prune
docker system prune
```

注意：`prune` 会删除未使用资源。执行前要确认是否有仍然需要的数据，尤其是 volume。

## 四、Dockerfile

### 1. 基本示例

以一个 Node.js 应用为例：

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

构建镜像：

```bash
docker build -t my-node-app:1.0.0 .
```

运行镜像：

```bash
docker run -d --name app -p 3000:3000 my-node-app:1.0.0
```

### 2. 常见指令

| 指令 | 作用 |
| --- | --- |
| `FROM` | 指定基础镜像 |
| `WORKDIR` | 设置工作目录 |
| `COPY` | 复制本地文件到镜像中 |
| `ADD` | 复制文件，额外支持自动解压和 URL，不建议滥用 |
| `RUN` | 构建镜像时执行命令 |
| `CMD` | 容器启动时默认执行的命令 |
| `ENTRYPOINT` | 容器启动入口命令 |
| `ENV` | 设置环境变量 |
| `ARG` | 设置构建参数 |
| `EXPOSE` | 声明容器监听端口 |
| `VOLUME` | 声明挂载点 |
| `USER` | 指定运行用户 |
| `HEALTHCHECK` | 定义健康检查 |

### 3. CMD 和 ENTRYPOINT

`CMD` 提供默认启动命令，可以被 `docker run` 后面的命令覆盖。

```dockerfile
CMD ["node", "server.js"]
```

`ENTRYPOINT` 更适合固定入口程序，`CMD` 可以作为默认参数。

```dockerfile
ENTRYPOINT ["node"]
CMD ["server.js"]
```

实际执行相当于：

```bash
node server.js
```

### 4. 构建上下文

执行：

```bash
docker build -t app .
```

最后的 `.` 表示构建上下文。Docker 会把上下文中的文件发送给 Docker 引擎，再根据 Dockerfile 构建镜像。

因此应该使用 `.dockerignore` 排除不需要的文件：

```gitignore
node_modules
.git
.env
dist
coverage
*.log
```

不要把敏感文件、依赖缓存、大型构建产物放进构建上下文。

### 5. 镜像分层和缓存

Dockerfile 中每条指令通常会生成一层镜像层。Docker 构建时会复用缓存。

优化示例：

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

这样做的原因：

- 依赖文件不变时，`npm ci` 这一层可以复用缓存。
- 业务代码频繁变化时，只会重新执行后面的复制和构建步骤。

### 6. 多阶段构建

多阶段构建可以把编译环境和运行环境分开，减少最终镜像体积。

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:1.25-alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

优点：

- 最终镜像不包含源代码依赖和构建工具。
- 镜像更小。
- 攻击面更小。

### 7. Dockerfile 最佳实践

- 使用明确版本的基础镜像，不随意使用 `latest`。
- 优先使用官方镜像或可信镜像。
- 用 `.dockerignore` 减少构建上下文。
- 把依赖安装步骤放在代码复制之前，提高缓存命中率。
- 使用多阶段构建减少镜像体积。
- 不要把密码、密钥、Token 写入镜像。
- 容器内应用日志输出到 stdout/stderr。
- 尽量使用非 root 用户运行应用。
- 每个容器只运行一个主要进程。
- 对关键服务增加健康检查。

## 五、数据持久化

### 1. 三种挂载方式

| 类型 | 说明 | 常见用途 |
| --- | --- | --- |
| volume | Docker 管理的数据卷 | 数据库数据、持久化文件 |
| bind mount | 宿主机路径挂载到容器 | 本地开发、配置文件挂载 |
| tmpfs | 数据只存在内存中 | 临时敏感数据、缓存 |

### 2. volume 示例

创建 volume：

```bash
docker volume create mysql-data
```

使用 volume：

```bash
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v mysql-data:/var/lib/mysql \
  mysql:8
```

查看 volume：

```bash
docker volume ls
docker volume inspect mysql-data
```

### 3. bind mount 示例

```bash
docker run -d --name nginx \
  -p 8080:80 \
  -v "$PWD/html:/usr/share/nginx/html" \
  nginx
```

bind mount 会直接使用宿主机目录。开发时很方便，但生产环境要注意权限和路径稳定性。

## 六、Docker 网络

### 1. 查看网络

```bash
docker network ls
docker network inspect bridge
```

### 2. 创建自定义网络

```bash
docker network create app-net
```

运行容器并加入网络：

```bash
docker run -d --name redis --network app-net redis:7
docker run -d --name app --network app-net my-app:1.0.0
```

在同一个自定义网络中，容器可以通过容器名互相访问。

例如应用连接 Redis：

```text
redis://redis:6379
```

这里的 `redis` 是容器名，也是 Docker 内部 DNS 可以解析的主机名。

### 3. 端口映射

```bash
docker run -p 8080:80 nginx
```

含义：

```text
宿主机端口:容器端口
```

访问宿主机 `8080` 端口，会转发到容器的 `80` 端口。

如果只写：

```bash
docker run -p 80 nginx
```

Docker 会随机分配宿主机端口。

## 七、Docker Compose

Docker Compose 用来定义和运行多容器应用。它通常通过 `compose.yaml` 或 `docker-compose.yml` 描述服务、网络、数据卷和环境变量。

### 1. 示例

```yaml
services:
  web:
    image: nginx:1.25-alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  redis-data:
```

启动：

```bash
docker compose up -d
```

查看：

```bash
docker compose ps
docker compose logs -f
```

停止并删除容器：

```bash
docker compose down
```

如果要同时删除 volume：

```bash
docker compose down -v
```

### 2. 常用 Compose 命令

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs -f
docker compose exec web sh
docker compose build
docker compose pull
docker compose restart
```

### 3. depends_on 的注意点

`depends_on` 只能控制启动顺序，不一定代表依赖服务已经完全可用。

例如数据库容器启动了，但数据库服务可能还在初始化。更稳妥的方式是：

- 给依赖服务加 `healthcheck`。
- 应用启动时实现重试连接。
- 在部署平台中使用健康检查和就绪检查。

## 八、镜像发布

### 1. 打标签

```bash
docker tag my-app:1.0.0 username/my-app:1.0.0
docker tag my-app:1.0.0 username/my-app:latest
```

### 2. 登录并推送

```bash
docker login
docker push username/my-app:1.0.0
docker push username/my-app:latest
```

### 3. 镜像标签策略

常见标签：

- `v1.0.0`：语义化版本。
- `1.0`：次版本。
- `latest`：最新稳定版本。
- `git-commit-sha`：和具体代码提交绑定。
- `dev`、`staging`、`prod`：环境标签，不建议作为唯一版本依据。

生产环境最好使用不可变标签，例如 Git commit SHA 或明确版本号，方便回滚和审计。

## 九、生产环境注意事项

### 1. 重启策略

```bash
docker run -d --restart unless-stopped nginx
```

常见策略：

| 策略 | 含义 |
| --- | --- |
| `no` | 默认，不自动重启 |
| `always` | 总是自动重启 |
| `unless-stopped` | 非手动停止时自动重启 |
| `on-failure` | 异常退出时重启 |

### 2. 资源限制

```bash
docker run -d --name app \
  --memory 512m \
  --cpus 1.5 \
  my-app:1.0.0
```

避免单个容器占满宿主机资源。

### 3. 健康检查

Dockerfile 示例：

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

健康检查可以帮助部署系统判断容器是否真的可用。

### 4. 安全建议

- 不要在镜像中保存密钥。
- 不要把 Docker socket 随意挂载进容器。
- 尽量使用非 root 用户。
- 只开放必要端口。
- 定期更新基础镜像。
- 对镜像做漏洞扫描。
- 使用只读文件系统或最小权限挂载。
- 使用 `.dockerignore` 避免把 `.env`、私钥、源码历史打进镜像。

## 十、常见问题排查

### 1. 容器启动后立刻退出

查看状态：

```bash
docker ps -a
```

查看日志：

```bash
docker logs container-name
```

常见原因：

- 启动命令错误。
- 应用配置缺失。
- 端口被占用。
- 依赖服务不可用。
- 程序运行后直接结束。

### 2. 端口无法访问

检查方向：

- 容器是否运行：`docker ps`
- 端口是否映射：`docker ps` 的 `PORTS` 字段
- 应用是否监听 `0.0.0.0`，而不是只监听 `127.0.0.1`
- 宿主机防火墙是否放行
- 访问的宿主机端口是否正确

### 3. 容器之间无法通信

检查方向：

- 是否在同一个 Docker network。
- 是否使用了正确的容器名或服务名。
- 目标服务端口是否正确。
- 服务是否已经启动完成。

### 4. 数据丢失

常见原因：

- 数据写在容器内部，没有挂载 volume。
- 执行了 `docker compose down -v` 删除了 volume。
- 挂载路径写错，导致数据写到别的位置。

数据库类服务必须确认数据目录已经挂载到 volume。

### 5. 磁盘空间不足

查看占用：

```bash
docker system df
```

清理未使用资源：

```bash
docker system prune
```

如果要清理未使用 volume：

```bash
docker volume prune
```

执行前要确认不会删除仍然需要的数据。

## 十一、常用工作流

### 1. 本地开发

```text
编写代码
  -> 编写 Dockerfile
  -> docker build 构建镜像
  -> docker run 本地验证
  -> docker compose 管理多服务环境
```

### 2. 部署应用

```text
提交代码
  -> CI 构建镜像
  -> 运行测试
  -> 推送镜像仓库
  -> 服务器拉取镜像
  -> 启动或滚动更新容器
```

### 3. 学习重点

- 理解镜像和容器的区别。
- 熟悉 `docker run`、`docker ps`、`docker logs`、`docker exec`。
- 会写基础 Dockerfile。
- 会使用 volume 保存数据。
- 会使用网络连接多个容器。
- 会用 Docker Compose 编排本地服务。
- 知道如何查看日志、端口、网络和磁盘占用。
