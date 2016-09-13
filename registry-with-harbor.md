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

### 配置 Harbor
---

1) 克隆harbor仓库

```
git clone https://github.com/vmware/harbor ~/harbor
```
 
2) 编辑配置

```
vim ~/harbor/Deploy/harbor.cfg
```
模板如下：
```
## Configuration file of Harbor

# 下面输入你的仓库网址，比如“reg.example.com”
# 不要使用 localhost 或者 127.0.0.1 作为 hostname
# 否则别人无法访问这个仓库。
hostname = docker.rd.mt

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = https

#Email account settings for sending out password resetting emails.
#这里的设置可以无视，只有密码找回才会用到。
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false

## The password of Harbor admin, change this before any production use
## 下面输入你的管理员密码。
harbor_admin_password = 123456

##By default the auth mode is db_auth, i.e. the credentials are stored in a local database.
#Set it to ldap_auth if you want to verify a user's credentials against an LDAP server.
# 使用公司内部ldap认证登录
auth_mode = ldap_auth

# The url for an ldap endpoint

ldap_url = ldap://192.168.170.13

#The basedn template to look up a user in LDAP and verify the user's password.
# For AD server, uses this template

ldap_basedn = mailtechgz\\%s

#The password for the root user of mysql db, change this before any production use.
db_password = root123

# Turn on or off the self-registration feature
# 是否开放注册。
self_registration = on

#Determine whether the UI should use compressed js files. 
#For production, set it to on. For development, set it to off.
use_compressed_js = on

#Maximum number of job workers in job service  
max_job_workers = 3 

#The expiration of token used by token service, default is 30 minutes
token_expiration = 30

#Determine whether the job service should verify the ssl cert when it connects to a remote registry.
#Set this flag to off when the remote registry uses a self-signed or untrusted certificate.
verify_remote_cert = on

#Determine whether or not to generate certificate for the registry's token.
#If the value is on, the prepare script creates new root cert and private key 
#for generating token to access the registry. If the value is off, a key/certificate must 
#be supplied for token generation.
customize_crt = on

# Information of your organization for certificat
# 用到 SSL ，这里填个人信息
crt_country = CN
crt_state = GuangZhou
crt_location = CN
crt_organization = Coremail
crt_organizationalunit = Mailtech
crt_commonname = docker.rd.mt
crt_email = me@rd.mt
#####
```

3) 安装准备, 生成SSL证书

```
 ./prepare
Generated configuration file: ./config/ui/env
Generated configuration file: ./config/ui/app.conf
Generated configuration file: ./config/registry/config.yml
Generated configuration file: ./config/db/env
Generated configuration file: ./config/jobservice/env
Clearing the configuration file: ./config/ui/private_key.pem
Clearing the configuration file: ./config/registry/root.crt
Generated configuration file: ./config/ui/private_key.pem
Generated configuration file: ./config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.
```

4) 配置 Nginx
经过上一步之后， config 目录下如下，注意 SSL 证书位置。
```
├── db
│   └── env
├── jobservice
│   ├── app.conf
│   └── env
├── nginx
│   ├── cert
│   ├── nginx.conf
│   ├── nginx.conf.bak
│   └── nginx.https.conf
├── registry
│   ├── config.yml
│   └── root.crt           ## CRT 证书
└── ui
    ├── app.conf
    ├── env
    └── private_key.pem    ## PEM 证书

6 directories, 11 files
```
进入 Nginx 目录，配置 Nginx, 移动你的证书到 cert/
```
cd ~/harbor/Deploy/config/nginx
cp ~/harbor/Deploy/config/registry/root.crt ~/harbor/Deploy/config/nginx/cert/
cp ~/harbor/Deploy/config/ui/private_key.pem ~/harbor/Deploy/config/nginx/cert/ 
```

备份一下原文件，使用https配置
```
mv nginx.conf nginx.conf.bak && cp nginx.https.conf nginx.conf
```
修改 nginx.conf，要改的地方很少，如下

```
 ...

  server {
    listen 443 ssl;
    # 下面改成你的域名
    server_name docker.rd.mt;

    # SSL
    # 使用 ./prepare 指令生成证书就这样填。
    ssl_certificate /etc/nginx/cert/root.crt;
    ssl_certificate_key /etc/nginx/cert/private_key.pem;

    ...

  server {
    listen 80;
    server_name docker.rd.mt;
    location / {
          proxy_pass http://ui/;
          proxy_set_header Host $http_host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    
          # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
          proxy_set_header X-Forwarded-Proto $scheme;
    
          proxy_buffering off;
          proxy_request_buffering off;
    }
    location /service/ {
          proxy_pass http://ui/service/;
          proxy_set_header Host $http_host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
          # When setting up Harbor behind other proxy, such as an Nginx instance, remove the below line if the proxy already has similar settings.
          proxy_set_header X-Forwarded-Proto $scheme;
        
          proxy_buffering off;
          proxy_request_buffering off;
    }    
        
   
    }


    ...
```

这里使用了 80 端口、5000 端口以及 443 端口，你需要确保这些端口没有占用
如果你需要修改端口，要注意 docker-compose.yml 的修改，不要只在 nginx.conf 中修改

ldap登录认证时，调用openldap 里 SearchAll方法时会出现类似 [CM-27219](http://jira.mailtech.cn/browse/CM-27219) 的问题，
这个是一个openldap库使用的坑来的
需要改一下openldap库的源码， ```harbor/vendor/github.com/mqu/openldap/options-errors.go```
插入以下代码段
```
func (self *Ldap) SetOptionPointer(opt int, value unsafe.Pointer) error {

    // API: ldap_set_option (LDAP *ld,int option, LDAP_CONST void *invalue));
    rv := C.ldap_set_option(self.conn, C.int(opt), value)

    if rv == LDAP_OPT_SUCCESS {
        return nil
    }

    return errors.New(fmt.Sprintf("LDAP::SetOption() error (%d) : %s", int(rv), ErrorToString(int(rv))))
}
```

修改 ```harbor/auth/ldap/ldap.go```文件
```
//在 ldap.SetOption(openldap.LDAP_OPT_PROTOCOL_VERSION, openldap.LDAP_VERSION3) 下一行插入

ldap.SetOptionPointer(openldap.LDAP_OPT_REFERRALS,nil)

// searchAll的filter也要修改为

filter := "(sAMAccountName=" + m.Principal + ")"
ldap.SearchAll("dc=mailtech,dc=local", scope, filter, attributes)

```

5) 构建运行 Harbor
```
$ cd ~/harbor/Deploy
$ docker-compose up
```

6) docker push 

登录前需把certs下的 ```root.crt``` 拷贝到client对应目录 ```/etc/docker/docker.rd.mt/ca.crt```下，再重启docker
用户登录成功后用docker push命令向 Harbor 推送一个Docker image：
要先在web客户端下新建一个属于自己的project 如下面例子的 ```demo```


```
docker tag helloworld docker.rd.mt/demo/helloworld 
docker push docker.rd.mt/demo/helloworld
```
push 过后，就可以登录Harbor界面，在私有项目```demo```下看到刚push的 ```helloworld``` 镜像

 1. 首先，docker 客户端会重复login的过程，首先发送请求到registry,之后得到token 服务的地址；
 2. 之后，Docker 客户端在访问ui容器上的token服务时会提供额外信息，指明它要申请一个对image library/hello-world进行push操作的token；
 3. token 服务在经过Nginx转发得到这个请求后，会访问数据库核实当前用户是否有权限对该image进行push。如果有权限，它会把image的信息以及push动作进行编码，并用私钥签名，生成token返回给Docker客户端；
 4. 得到token之后 Docker客户端会把token放在请求头部，向registry发出请求，试图开始推送image。 Registry 收到请求后会用公钥解码token并进行核对，一切成功后，image的传输就开始了。


### 参考文献
> 1. [Configuring Harbor with HTTPS Access] (https://github.com/vmware/harbor/blob/master/docs/configure_https.md)
> 2. [搭建 Docker Registry(V2)](https://zuolan.me/p/build-docker-registry.html)
> 3. [企业级Registry开源项目Harbor架构简介](http://dockone.io/article/1179)








 
 
 