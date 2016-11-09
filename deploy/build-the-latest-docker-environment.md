## Docker源码编译安装

### 源码编译安装之前
Docker源码需在docker容器编译环境中编译，所以先安装docker，安装环境依旧是ubuntu16.04

1). 从 [官方下载地址](https://golang.org/dl/) 下载最新稳定版本:
  
```
    安装 Golang
    
    #Golang
    wget https://storage.googleapis.com/golang/go1.7.3.linux-amd64.tar.gz  
    sudo tar -xzf go1.7.3.linux-amd64.tar.gz -C /usr/local
```

2). 下载最新的docker源码:

```
go get -d -u github.com/docker/docker
package github.com/docker/docker: no buildable Go source files in /go/src/github.com/docker/docker
```

3). 本地编译
源的问题放在国内基本是被墙的，建议替换镜像源

```
Dockerfile中的
ARG APT_MIRROR=httpredir.debian.org
替换成
ARG APT_MIRROR=mirrors.ustc.edu.cn

其他的git托管的package文件也可能是无法访问的， 
 
建议先本地下载回来， 替换成本地服务器的url(用 `python -m SimpleHttpServer` 提供内网下载) 

```

编译流程
```
cd $GOPATH/src/github.com/docker/docker/

# as root (or prepend with sudo)
make build 
# this takes a while and prints the output from "Docker build ." 
# It ends with: "Successfully built ..".

# as root (or prepend with sudo)
make binary

# This runs the image we created with "make build". 
# Ends with "Created binary: bundles/1.8.0-dev/binary/docker-1.8.0-dev",
# where "1.8.0-dev" is the version I'm building, 
# the next unreleased version at the time of writing.
            
```
生成的 docker bin文件在 ` $GOPATH/src/github.com/docker/docker/bundles/latest/` 目录下

### 参考文献
> 1. [build-the-newest-docker-environment] (http://nanxiao.me/en/build-the-newest-docker-environment/)
> 2. [Docker源码编译安装](http://blog.csdn.net/lwyeluo/article/details/51765309)
