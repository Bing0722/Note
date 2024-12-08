## DOcker 学习笔记

Docker 是一个开源的容器化平台，用于创建、管理、部署容器化应用程序。以下是一些常用的 Docker 指令，帮助你管理容器、镜像、网络和卷等资源。

### docker 镜像操作

- 拉取镜像
```bash
docker pull [镜像名称]:[版本号]
```

- 查看本地镜像
```bash
docker images
```

- 删除镜像
```bash
docker rmi [镜像名称]:[版本号]
```

- 构建镜像
```bash
docker build -t [镜像名称]:[版本号] [Dockerfile路径]
```

### docker 容器操作

- 运行容器
```bash
docker run -d -p [宿主机端口]:[容器端口] --name [容器名称] [镜像名称]:[版本号]
```
常用参数：
- -d：后台运行容器
- -p：将容器的端口映射到宿主机
- --name：指定容器名称
- -it：以交互模式运行容器
- -v：将宿主机目录挂载到容器
- --rm：容器退出后自动删除
例如：
```bash
docker run -d -p 8080:80 --name my-nginx nginx:latest
```

- 查看运行中的容器
```bash
docker ps
```

- 查看所有容器
```bash
docker ps -a
```

- 启动容器
```bash
docker start [容器名称或ID]
```

- 停止容器
```bash
docker stop [容器名称或ID]
```

- 删除容器
```bash
docker rm [容器名称或ID]
```

- 查看容器日志
```bash
docker logs [容器名称或ID]
```

- 进入运行中的容器
```bash
docker exec -it [容器名称或ID] /bin/bash
```

### docker 网络操作

- 查看网络
```bash
docker network ls
```

- 创建网络
```bash
docker network create [网络名称]
```

- 连接容器到网络
```bash
docker network connect [网络名称] [容器名称或ID]
```

- 断开容器与网络的连接
```bash
docker network disconnect [网络名称] [容器名称或ID]
```

- 删除网络
```bash
docker network rm [网络名称]
```

### docker 卷操作

- 创建卷
```bash
docker volume create [卷名称]
```

- 查看卷
```bash
docker volume ls
```

- 删除卷
```bash
docker volume rm [卷名称]
```

- 挂载卷
```bash
docker run -d -v [卷名称]:[容器内目录] --name [容器名称] [镜像名称]:[版本号]
```

### docker 其他操作

- 查看 docker 信息
```bash
docker info
```

- 查看 docker 事件
```bash
docker events
```

- 清理未使用的资源
```bash
docker system prune
```

- 导出容器为 tar 文件
```bash
docker export [容器名称或ID] > [文件名.tar]
```

- 导入 tar 文件为镜像
```bash
cat [文件名.tar] | docker import - [镜像名称]:[版本号]
```

- 保存镜像为 tar 文件
```bash
docker save -o [文件名.tar] [镜像名称]:[版本号]
```

- 从 tar 文件加载镜像
```bash
docker load -i [文件名.tar]
```

### Docker Compose
Docker Compose 是 Docker 官方编排（Orchestration）工具，用于定义和运行多容器 Docker 应用。

- 启动服务
```bash
docker-compose up
```

- 停止服务
```bash
docker-compose stop
```

- 后台启动服务
```bash
docker-compose up -d
```
- 重新构建服务
```bash
docker-compose build
```