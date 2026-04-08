docker常用命令总结：
``` shell
docker run -it ubuntu bash
```
 解释：

	run：创建并启动容器
	-it：交互模式（能输入命令）
	ubuntu：镜像
	bash：启动 shell
	-name: 可选，给容器命名方便管理

效果：  
你直接进入一个“临时 Linux”，直接exit停止容器但是不删除

拉镜像:
```shell
docker pull ubuntu
```

查看本地镜像：
```shell
docker images
```

删除镜像：
```shell
docker rmi ubuntu
```

查看正在运行的容器：
```shell
docker ps
```

查看所有容器：

```shell
docker ps -a
```

启动一个已经存在的容器：
```shell
docker start mylinux
```

停止容器：
```shell
docker stop mylinux
```

删除容器：
```shell
docker rm mylinux
```

重新进入容器（重点）：
```shell
docker exec -it mylinux bash
```

挂载本地目录（重点）：
```shell
docker run -it -v $(pwd):/app ubuntu bash
```

映射端口：
```shell
docker run -p 8080:80 nginx
```

后台运行（daemon）：
```shell
docker run -d nginx
```

查看容器日志：
```shell
docker logs mylinux
```

实时日志：
```shell
docker logs -f mylinux
```

查看容器详情：
```shell
docker inspect mylinux
```

删除所有停止容器：
```shell
docker container prune
```

删除无用镜像：
```shell
docker image prune
```

一次性容器：
```shell
docker run -it --rm ubuntu bash
```

### Dockerfile

| 指令           | 功能                                 |
| ------------ | ---------------------------------- |
| `FROM`       | 指定基础镜像，比如 `ubuntu` 或 `golang:1.21` |
| `WORKDIR`    | 设置工作目录，类似 `cd`                     |
| `COPY`       | 把本地文件复制到镜像里                        |
| `ADD`        | 类似 `COPY`，可以解压 tar 或从 URL 下载       |
| `RUN`        | 在镜像里执行命令，通常用于安装依赖                  |
| `CMD`        | 容器启动时执行的命令（可被 `docker run` 覆盖）     |
| `ENTRYPOINT` | 容器启动入口，不易被覆盖，更固定                   |
| `ENV`        | 设置环境变量                             |
| `EXPOSE`     | 声明容器运行的端口（只是声明，不会映射端口）             |
示例1：

```dockerfile
# 1. 选择基础镜像
FROM node:20-alpine

# 2. 设置工作目录
WORKDIR /app

# 3. 复制依赖文件并安装
COPY package*.json ./
RUN npm install

# 4. 复制代码
COPY . .

# 5. 容器启动命令
CMD ["node", "index.js"]

# 6. 声明端口（可选）
EXPOSE 3000

```

示例2:

``` dockerfile
 # 1. 选择基础镜像
FROM node:20-alpine

# 2. 设置工作目录
WORKDIR /app

# 3. 复制依赖文件并安装
COPY package*.json ./
RUN npm install

# 4. 复制代码
COPY . .

# 5. 容器启动命令
CMD ["node", "index.js"]

# 6. 声明端口（可选）
EXPOSE 3000

 ``` 

小技巧：
- **顺序优化**：频繁变化的文件放在后面 COPY，这样可以利用缓存加速构建。
- **减少镜像体积**：用轻量基础镜像（`alpine`）、多阶段构建。
- **避免 `RUN apt-get upgrade`**，因为容易打破缓存层。
- **尽量少用 root 权限**，可用 `USER` 指令切换用户。