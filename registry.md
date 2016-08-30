## 安装 Docker 
系统的要求跟 Ubuntu 情况类似，64 位操作系统，内核版本至少为 3.10。
```
sudo curl -sSL https://get.docker.com/ | sh
```

##  安装Docker compose 
```
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

## 安装 openssl与httpd-tools
openssl用于生成证书, httpd-tools用于生成认证账号和密码
```
yum install -y openssl openssl-devel httpd-tools
```

## 证书生成步骤

1. 创建证书存放目录
```
  mkdir -p /docker-registry-compose/certs
```
2. 生成证书密钥
```
openssl req -nodes -subj "/C=CN/ST=GuangDong/L=GuangZhou/CN=docker.rd.mt" -newkey rsa:4096 -keyout docker-registry.key -out docker-registry.csr
```
3. 生成证书
```
openssl x509 -req -days 3650 -in docker-registry.csr -signkey docker-registry.key -out docker-registry.crt
```
4. 导入证书
```
mkdir /etc/docker/certs.d/docker.rd.mt
cp docker-registry.crt /etc/docker/certs.d/docker.rd.mt/ca.crt
```
5. 重启docker
```
service restart docker
```
6. 生成http password
新建用户 账号名：admin 密码：admin  
该账号用于登陆Docker-Registry
```
htpasswd -cb httppwd admin admin
```

## 启动相关容器
当前目录信息
```
 [root@localhost docker-registry-compose]# ll
   total 8
   drwxr-xr-x. 2 root root  84 Aug 28 23:23 certs
   -rw-r--r--. 1 root root 512 Aug 29 00:03 docker-compose.yml
   -rw-r--r--. 1 root root  44 Aug 29 00:04 httppwd

```
在/docker-registry-compose/ 目录下新建 docker-compose.yml文件，

```
   
docker-registry:
  container_name: mt-registry
  image: registry:2
  ports:
    - 5000:5000
  volumes:
    - /data:/tmp/registry-dev

docker-registry-proxy:
  container_name: mt-registry-proxy
  image: containersol/docker-registry-proxy
  ports:
    - 443:443
  environment:
    SERVER_NAME: "0.0.0.0"
    REGISTRY_PORT: "5000"
    REGISTRY_HOST: "docker-registry"
  links:
    - docker-registry:docker-registry
  volumes:
    - ./httppwd:/etc/nginx/.htpasswd:ro
    - ./certs:/etc/nginx/ssl:ro

```
启动容器
```
  docker-compose up -d
```

## 测试  `For Linux 客户端`

1. 登录registry
登录前需把certs下的docker-registry.crt 拷贝到client对应目录下，再重启docker
```
mkdir -p /etc/docker/certs.d/docker.rd.mt
cp docker-registry.crt /etc/docker/certs.d/docker.rd.mt/ca.crt
service docker restart
```
client登录
```
docker login -u admin -p admin docker.rd.mt
```
 显示Login Succeeded说明配置成功

2. 推送镜像 
```
docker tag helloworld docker.rd.mt/helloworld
docker push docker.rd.mt/helloworld
```

## 测试  `For Mac版客户端`

1. 进入Mac的Docker所在虚拟机
终端如下命令
```
screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
```
进入Moby Linux虚拟机命令行登录界面
在moby login:后，直接输入root回车，则可登录到docker所在的虚拟机；
2. 添加CA证书
把.crt证书放到docker所在虚拟机的共享目录中（该目录可通过Mac顶栏Docker图标右击菜单>Preferences 设置对话框中，选择File Sharing页签，可以查看编辑共享目录）
并且copy到/etc/docker/certs.d/docker.rd.mt文件夹下（该文件夹需自行建立）
3.  重启虚拟机内docker服务 
```
service docker restart
```


