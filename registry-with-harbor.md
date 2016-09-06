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

# ����������Ĳֿ���ַ�����硰reg.example.com����
# ��Ҫʹ�� localhost ���� 127.0.0.1 ��Ϊ hostname��
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
auth_mode = db_auth

# The url for an ldap endpoint
# ldap���Բ��
ldap_url = ldaps://ldap.mydomain.com

#The basedn template to look up a user in LDAP and verify the user's password.
# For AD server, uses this template
# �ҵĵ������� docker.rd.mt�� ������������ dc=rd,dc=mt:
#ldap_basedn = CN=%s,OU=Dept1,DC=mydomain,DC=com
ldap_basedn = uid=%s,ou=people,dc=rd,dc=mt

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
    rewrite ^/(.*) https://$server_name$1 permanent;

    ...
```

����ʹ���� 80 �˿ڡ�5000 �˿��Լ� 443 �˿ڣ�����Ҫȷ����Щ�˿�û��ռ�ã�  
�������Ҫ�޸Ķ˿ڣ�Ҫע�� docker-compose.yml ���޸ģ���Ҫֻ�� nginx.conf ���޸�

5) �������� Harbor
```
$ cd ~/harbor/Deploy
$ docker-compose up
```

6) docker push 

��¼ǰ���certs�µ�docker-registry.crt ������client��ӦĿ¼�£�������docker
�û���¼�ɹ�����docker push������Harbor ����һ��Docker image��
docker push docker.rd.mt/library/hello-world

 a) ���ȣ�docker �ͻ��˻��ظ�login�Ĺ��̣����ȷ�������registry,֮��õ�token ����ĵ�ַ��
 b) ֮��Docker �ͻ����ڷ���ui�����ϵ�token����ʱ���ṩ������Ϣ��ָ����Ҫ����һ����image library/hello-world����push������token��
 c) token �����ھ���Nginxת���õ��������󣬻�������ݿ��ʵ��ǰ�û��Ƿ���Ȩ�޶Ը�image����push�������Ȩ�ޣ������image����Ϣ�Լ�push�������б��룬����˽Կǩ��������token���ظ�Docker�ͻ��ˣ�
 d) �õ�token֮�� Docker�ͻ��˻��token��������ͷ������registry����������ͼ��ʼ����image�� Registry �յ��������ù�Կ����token�����к˶ԣ�һ�гɹ���image�Ĵ���Ϳ�ʼ�ˡ�


### �ο�����
> 1. [Configuring Harbor with HTTPS Access] (https://github.com/vmware/harbor/blob/master/docs/configure_https.md)
> 2. [� Docker Registry(V2)](https://zuolan.me/p/build-docker-registry.html)
> 3. [��ҵ��Registry��Դ��ĿHarbor�ܹ����](http://dockone.io/article/1179)








 
 
 