## Shipyard本地重新编译源码安装方式

## 源码编译安装

 1. Shipyard的源码托管在 [Github](https://github.com/shipyard/shipyard)  
    在本地开了分支做了UI界面汉化及LDAP用户登录认证 [Gitlab](http://git.mailtech.cn/zfmai/shipyard)  

 2. Shipyard整个项目是通过go语言实现核心逻辑部分  
    也就是说是通过shipard-controller(go实现)调用 Docker Remote API  
    使用angular + semantic 实现的UI部分  
    这两部分合起来构成了Shipyard 的web管理系统  
 
## 项目重新构建流程 
 
  1.1 从 [官方下载地址](https://golang.org/dl/) 下载最新稳定版本:
  
```
    安装 Golang 和 Node.js(npm for bower to build the Angular frontend)
    
    #Golang
    
    wget https://storage.googleapis.com/golang/go1.4.linux-amd64.tar.gz  
    sudo tar -xzf go1.4.linux-amd64.tar.gz -C /usr/local
    
    #Node.js 
    
    curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
    sudo apt-get install -y nodejs
    
```  
  1.2 打开HOME目录下的.bashrc文件
```
vim ~/.bashrc

# 添加下面的内容 

export GOROOT=/usr/local/go
export GOBIN=$GOROOT/bin
export PATH=$GOROOT/bin:$PATH
export GOPATH=$HOME/workspace/Go
```
1.3 保存设置并运行命令使其生效
```
source ~/.bashrc
```
1.4 安装godep工具(管理go项目相关的依赖), bower(管理前端项目相关的依赖) 

```
go get github.com/tools/godep

npm install bower -g
```
1.5 下载源码  
go开发环境在 $HOME/workspace/Go下，src是放源码的目录，下面有github, gopkg.org等目录等,在第三方网站下载的源码

```
mkdir -p $HOME/workspace/Go/src/github/shipyard/
cd $HOME/workspace/Go/src/github/shipyard/
git clone http://git.mailtech.cn/zfmai/shipyard.git
```

1.6 编译

```
# controller部分
make build

# UI部分 
make media

# 构建shipyard-controller对应的镜像
make image 
```

1.7 部署

参考deploy的脚本，启动的参数都已经在脚本中指定了，如ldap服务器的地址，默认新用户的权限

```
master节点本地部署
./deploy.sh
```

销毁shipyard相关的容器
```
ACTION=remove  ./deploy.sh 
```

在集群(cluster)中添加其他节点，需要在其他节点开启etcd(服务发现的容器)

```
# 192.168.200.136 为master节点IP 
ACTION=node DISCOVERY=etcd://192.168.200.136:4001 ./deploy.sh 
```


          
      
    


