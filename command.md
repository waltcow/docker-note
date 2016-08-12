###  list images
```
docker image
```

###  To list all running and stopped containers

```
docker ps -a
```

### list all running containers
```
docker ps -a -f status=running
```

### show all containers
```
docker ps -a
```

### To remove all containers that are NOT running
```
docker rm `docker ps -aq -f status=exited`
```

### To remove image
```
docker rmi
```

### start container
启动容器有两种方式，一种是基于镜像新建一个容器并启动，
另外一个是将在终止状态（stopped）的容器重新启动。
```
 docker run ubuntu:14.04 /bin/echo 'Hello world'
```
下面的命令则启动一个 bash 终端，允许用户进行交互

```
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@af8bae53bdd3:/#
```
其中， `-t`  选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入
上， `-i` 则让容器的标准输入保持打开。

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：
检查本地是否存在指定的镜像，不存在就从公有仓库下载

- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止