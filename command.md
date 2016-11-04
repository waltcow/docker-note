###  list images
```
docker image
```
### docker ps usage 
```
Usage:	docker ps [OPTIONS]

List containers

  -a, --all          Show all containers (default shows just running)
  -f, --filter=[]    Filter output based on conditions provided
  --format           Pretty-print containers using a Go template
  --help             Print usage
  -l, --latest       Show the latest created container (includes all states)
  -n=-1              Show n last created containers (includes all states)
  --no-trunc         Don't truncate output
  -q, --quiet        Only display numeric IDs
  -s, --size         Display total file sizes
```

###  To list all running and stopped containers


```
-a, --all          Show all containers (default shows just running)

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

## 如何进入正在执行的 docker container 
docker1.3增加新的exec命令行工具，进入container更加方便，简单说命令是：“docker exec -i -t bash”
```
$ docker run -i -t -d ubuntu bash
740e78a3406f72db08973c2db6c2e60286504135d48ab5d431b706a342b051bd
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
740e78a3406f        ubuntu:14.04        "bash"              4 seconds ago       Up 3 seconds                            cocky_pike
$ docker exec -i -t 740e78a3406f bash
root@740e78a3406f:$ ls
```

## 使用squid加快docker rebuild的速度, 避免每次都要重新下载package
```
# get squid
docker run --name squid -d --restart=always \
  --publish 3128:3128 \
  --volume /var/spool/squid3 \
  sameersbn/squid:3.3.8-11

# optionally in another terminal run tail on logs
docker exec -it squid tail -f /var/log/squid3/access.log

# get squid ip to use in docker build
SQUID_IP=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' squid)

# build your instance
docker build --build-arg http_proxy=http://$SQUID_IP:3128 .
```

# Docker One Liner Examples

### Examples for automating Docker and general lazyness

```
# start shells in the following OS's (rm means it will delete when you exit the shell, see the mittens example for persistance):
docker run -it --rm ubuntu
docker run -it --rm debian
docker run -it --rm fedora
docker run -it --rm centos
docker run -it --rm busybox

# start a container named mittens that will run in the background
docker run -itd --name mittens ubuntu

# now attach to the shell of the newly created container by name (ID also valid)
docker attach mittens

# start a stopped container by ID or name
docker start <container_id or name>
docker start mittens 

# Stop a container by ID or name
docker stop <container_id or name>
docker stop mittens

# Start a stopped container by ID or name
docker start <container_id or name>
docker start mittens

# Attach to a running container.
docker attach <container_id or name>
docker attach mittens

# get json formatted detailed information about a container
docker inspect <container_id or name>
docker inspect mittens
# parse using grep for interesting fields like so to get a container IP address:
docker inspect mittens | grep IPAddress

# Docker inspect the last container created
docker inspect $(docker ps -qal)

# Docker inspect all containers
docker inspect $(docker ps -qa)

# Delete all images named <none> (untagged images)
delnone() { docker rmi $(docker images | grep none | awk '{print $3}') ;}

# Or another way cleanup all of those untagged images (Those labeled with <none>)
docker images -a
# REPOSITORY                          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
# <none>                              <none>              4008b117428e        17 hours ago        210.2 MB
# <none>                              <none>              53eb80109e88        17 hours ago        210.2 MB
# <none>                              <none>              f48d7a9838a0        17 hours ago        210.2 MB

# If a container is using it or there is a conflict it will abort the current image delta and move on to the next.
docker rmi $(docker images -a | grep "^<none>" | awk '{print $3}')

# Delete all containers matching the passed paramater; e.g. 'delimg foo' deletes all images named 'foo'
delimg() { docker rmi $(docker images | grep $@ | awk '{print $3}') ;}

# to remove a container    
docker rm  <container_id or name>

# to remove all containers, even stopped ones.   
docker rm $(docker ps -aq)

# to stop all containers.   
docker stop $(docker ps -q)

# view the last container to be started
docker ps -l

# Get the container ID column only
docker ps -l -q

# Get the container count of all running containers
docker ps -q | wc -l

# Get the container count of all running and stopped containers
docker ps -qa | wc -l

# stop the last container created
docker stop $(docker ps -l -q) 

# Stop and delete the last container created
docker rm -f `docker ps -ql`
# or
docker ps -l -q | xargs docker stop | xargs docker rm

# delete the container name
docker rm <container_id or name>

# Remove the last container created (running or stopped)
docker rm -f $(docker ps -ql)

# Remove all containers forcefully if they are running -f is nice (fastest)
docker rm -f `docker ps -qa`

# another way to stop and delete the last container created (-f above is still fastest/simplest)
docker ps -l -q | awk '{ print $1 }' | xargs docker stop | awk '{ print $1 }' | xargs docker rm

# remove all containers
docker rm $(docker ps -a -q)
#... or ...
docker ps -a -q | xargs docker rm

# List all network IDs only
docker network ls -q

# Inspect all networks
docker network inspect $(docker network ls -q)

# Delete all networks
docker network rm $(docker network ls -q)'

# When you 'docker run' an image and it fails at runtime, it will appear as Exited for example:"Exited (0) 8 days ago"
 # exiited containers are  refered to as "dangling" images.
 # delete all exited/dangling images
docker rmi $(docker images -q --filter "dangling=true")

# same as above but for containers, remove all Exited/failed containers.
docker rm $(docker ps -q -a  --filter "dangling=true")

 # bind a specific port to the container and host OS.
docker run  -i -t -p 81:80 --name container_name  image_name
80/tcp -> 0.0.0.0:81

# bind a random port to the container and host OS for the "Expose" binding in Dockerfile.
docker run  -i -t  -P --name container_name  image_name
docker port $(docker ps -l -q)

# remove a single image    
docker rmi <image_id>

# return all containers on the host    
docker ps

# stop all running containers (Note: simply replace "grep Up" with whatever column value you want to match on.
docker ps -a | grep Up | awk '{ print $1 }' | xargs docker stop

# delete all containers    
docker rm $(docker ps -a -q)

# delete all containers    
docker rm $(docker ps -a -q)

# image operations are nearly identical to container operations.
docker images | grep some_image_name | awk '{ print $3 }' | xargs docker rmi
 # or ...
docker rmi $(docker images | grep some_image_name | awk '{ print $3 }')
 # Or change the awk operation to the image "tag" column and parse an OS name for example
docker rmi $(docker images | grep centos | awk '{ print $2 }')

# stop and remove all running containers. similar approach due to the consistent API to containers as prior with images. (status=='Up')
docker ps -a | grep Up | awk '{ print $6 }' | xargs docker stop

#stop and delete running containers
docker ps -l -q | awk '{ print $1 }' | xargs docker stop | awk '{ print $1 }' | xargs docker rm

# start all stopped containers (after a reboot for example)
docker start $(docker ps -qa)

#Attach to a bash shell in the last started container
dockexecl() { docker exec -i -t $(docker ps -l -q) bash ;}

#Attach to a bash shell in the specified container ID passed to $ dockexecl <cid>
dockexec() { docker exec -i -t $@ bash ;}

#Get the IP address of all running  containers
docker inspect --format "{{ .NetworkSettings.IPAddress }}" $(docker ps -q)

#Get the IP address of the last started container
docker inspect --format "{{ .NetworkSettings.IPAddress }}" $(docker ps -ql)

# If the network is a network you created simply using grep on docker inspect is quick
$ docker network inspect  mcv1  | grep -i ipv4
       "IPv4Address": "192.168.1.106/24",

# Or look at the gateway of the network
$ docker network inspect  mcv1  | grep Gateway
       "Gateway": "192.168.1.1/24"

# Inspect and parse all IPs for all containers
$ docker inspect $(docker ps -qa) | grep IPA | grep [0-9]
         "IPAddress": "192.168.1.130",
         "IPAddress": "192.168.1.129",
         "IPAddress": "192.168.1.128",

# Example Docker network inspect all network subnets
docker network inspect $(docker network ls -q) | grep "Subnet\|Gateway"
         "Subnet": "172.17.0.0/16",
         "Gateway": "172.17.0.1"
         "Subnet": "172.16.86.0/24",
         "Gateway": "172.16.86.2/24"
         "Subnet": "192.168.1.0/24",
         "Gateway": "192.168.1.1/24"
                    

#stop and delete a container by name
docker stop <image_name> && docker rm flow_img

# Gracefully stop and delete all container
docker rm $(docker stop $(docker ps -aq))

# Kill and delete all containers
docker rm $(docker kill $(docker ps -aq))
```


