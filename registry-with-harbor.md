## ��װ Docker 
ϵͳ��Ҫ��� Ubuntu ������ƣ�64 λ����ϵͳ���ں˰汾����Ϊ 3.10��
```
sudo curl -sSL https://get.docker.com/ | sh
```

##  ��װDocker compose 
```
curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### ���� Harbor
---

1) ��¡harbor�ֿ�

```
git clone https://github.com/vmware/harbor ~/harbor
```
 
2) �༭����

```
vim ~/harbor/Deploy/harbor.cfg
```
ģ�����£�
```
## Configuration file of Harbor

# ����������Ĳֿ���ַ�����硰reg.example.com��
# ��Ҫʹ�� localhost ���� 127.0.0.1 ��Ϊ hostname
# ��������޷���������ֿ⡣
hostname = docker.rd.mt

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = https

#Email account settings for sending out password resetting emails.
#��������ÿ������ӣ�ֻ�������һزŻ��õ���
email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false

## The password of Harbor admin, change this before any production use
## ����������Ĺ���Ա���롣
harbor_admin_password = 123456

##By default the auth mode is db_auth, i.e. the credentials are stored in a local database.
#Set it to ldap_auth if you want to verify a user's credentials against an LDAP server.
# ʹ�ù�˾�ڲ�ldap��֤��¼
auth_mode = ldap_auth

# The url for an ldap endpoint

ldap_url = ldap://192.168.170.13

#The basedn template to look up a user in LDAP and verify the user's password.
# For AD server, uses this template

ldap_basedn = mailtechgz\\%s

#The password for the root user of mysql db, change this before any production use.
db_password = root123

# Turn on or off the self-registration feature
# �Ƿ񿪷�ע�ᡣ
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
# �õ� SSL �������������Ϣ
crt_country = CN
crt_state = GuangZhou
crt_location = CN
crt_organization = Coremail
crt_organizationalunit = Mailtech
crt_commonname = docker.rd.mt
crt_email = me@rd.mt
#####
```

3) ��װ׼��, ����SSL֤��

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

4) ���� Nginx
������һ��֮�� config Ŀ¼�����£�ע�� SSL ֤��λ�á�
```
������ db
��   ������ env
������ jobservice
��   ������ app.conf
��   ������ env
������ nginx
��   ������ cert
��   ������ nginx.conf
��   ������ nginx.conf.bak
��   ������ nginx.https.conf
������ registry
��   ������ config.yml
��   ������ root.crt           ## CRT ֤��
������ ui
    ������ app.conf
    ������ env
    ������ private_key.pem    ## PEM ֤��

6 directories, 11 files
```
���� Nginx Ŀ¼������ Nginx, �ƶ����֤�鵽 cert/
```
cd ~/harbor/Deploy/config/nginx
cp ~/harbor/Deploy/config/registry/root.crt ~/harbor/Deploy/config/nginx/cert/
cp ~/harbor/Deploy/config/ui/private_key.pem ~/harbor/Deploy/config/nginx/cert/ 
```

����һ��ԭ�ļ���ʹ��https����
```
mv nginx.conf nginx.conf.bak && cp nginx.https.conf nginx.conf
```
�޸� nginx.conf��Ҫ�ĵĵط����٣�����

```
 ...

  server {
    listen 443 ssl;
    # ����ĳ��������
    server_name docker.rd.mt;

    # SSL
    # ʹ�� ./prepare ָ������֤��������
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

����ʹ���� 80 �˿ڡ�5000 �˿��Լ� 443 �˿ڣ�����Ҫȷ����Щ�˿�û��ռ��
�������Ҫ�޸Ķ˿ڣ�Ҫע�� docker-compose.yml ���޸ģ���Ҫֻ�� nginx.conf ���޸�

ldap��¼��֤ʱ������openldap �� SearchAll����ʱ��������� [CM-27219](http://jira.mailtech.cn/browse/CM-27219) �����⣬
�����һ��openldap��ʹ�õĿ�����
��Ҫ��һ��openldap���Դ�룬 ```harbor/vendor/github.com/mqu/openldap/options-errors.go```
�������´����
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

�޸� ```harbor/auth/ldap/ldap.go```�ļ�
```
//�� ldap.SetOption(openldap.LDAP_OPT_PROTOCOL_VERSION, openldap.LDAP_VERSION3) ��һ�в���

ldap.SetOptionPointer(openldap.LDAP_OPT_REFERRALS,nil)

// searchAll��filterҲҪ�޸�Ϊ

filter := "(sAMAccountName=" + m.Principal + ")"
ldap.SearchAll("dc=mailtech,dc=local", scope, filter, attributes)

```

5) �������� Harbor
```
$ cd ~/harbor/Deploy
$ docker-compose up
```

6) docker push 

��¼ǰ���certs�µ� ```root.crt``` ������client��ӦĿ¼ ```/etc/docker/docker.rd.mt/ca.crt```�£�������docker
�û���¼�ɹ�����docker push������ Harbor ����һ��Docker image��
Ҫ����web�ͻ������½�һ�������Լ���project ���������ӵ� ```demo```


```
docker tag helloworld docker.rd.mt/demo/helloworld 
docker push docker.rd.mt/demo/helloworld
```
push ���󣬾Ϳ��Ե�¼Harbor���棬��˽����Ŀ```demo```�¿�����push�� ```helloworld``` ����

 1. ���ȣ�docker �ͻ��˻��ظ�login�Ĺ��̣����ȷ�������registry,֮��õ�token ����ĵ�ַ��
 2. ֮��Docker �ͻ����ڷ���ui�����ϵ�token����ʱ���ṩ������Ϣ��ָ����Ҫ����һ����image library/hello-world����push������token��
 3. token �����ھ���Nginxת���õ��������󣬻�������ݿ��ʵ��ǰ�û��Ƿ���Ȩ�޶Ը�image����push�������Ȩ�ޣ������image����Ϣ�Լ�push�������б��룬����˽Կǩ��������token���ظ�Docker�ͻ��ˣ�
 4. �õ�token֮�� Docker�ͻ��˻��token��������ͷ������registry����������ͼ��ʼ����image�� Registry �յ��������ù�Կ����token�����к˶ԣ�һ�гɹ���image�Ĵ���Ϳ�ʼ�ˡ�


### �ο�����
> 1. [Configuring Harbor with HTTPS Access] (https://github.com/vmware/harbor/blob/master/docs/configure_https.md)
> 2. [� Docker Registry(V2)](https://zuolan.me/p/build-docker-registry.html)
> 3. [��ҵ��Registry��Դ��ĿHarbor�ܹ����](http://dockone.io/article/1179)








 
 
 