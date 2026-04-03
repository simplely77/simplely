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