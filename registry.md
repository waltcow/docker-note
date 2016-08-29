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

## ��װ openssl��httpd-tools
openssl��������֤��, httpd-tools����������֤�˺ź�����
```
yum install -y openssl openssl-devel httpd-tools
```

## ֤�����ɲ���

1. ����֤����Ŀ¼
```
  mkdir -p /docker-registry-compose/certs
```
2. ����֤����Կ
```
openssl req -nodes -subj "/C=CN/ST=GuangDong/L=GuangZhou/CN=docker.rd.mt" -newkey rsa:4096 -keyout docker-registry.key -out docker-registry.csr
```
3. ����֤��
```
openssl x509 -req -days 3650 -in docker-registry.csr -signkey docker-registry.key -out docker-registry.crt
```
4. ����֤��
```
mkdir /etc/docker/certs.d/docker.rd.mt
cp docker-registry.crt /etc/docker/certs.d/docker.rd.mt/ca.crt
```
5. ����docker
```
service restart docker
```
6. ����http password
```
htpasswd -cb httppwd admin admin
(�˺�����admin ���룺admin)���˺����ڵ�½Docker-Registry
```

## �����������
��ǰĿ¼��Ϣ
```
 [root@localhost docker-registry-compose]# ll
   total 8
   drwxr-xr-x. 2 root root  84 Aug 28 23:23 certs
   drwxr-xr-x. 3 root root  19 Aug 26 15:18 data
   -rw-r--r--. 1 root root 512 Aug 29 00:03 docker-compose.yml
   -rw-r--r--. 1 root root  44 Aug 29 00:04 httppwd

```
��/docker-registry-compose/ Ŀ¼���½� docker-compose.yml�ļ���

```
   
docker-registry:
  container_name: mt-registry
  container_name: mt-registry
  image: registry:2
  ports:
    - 5000:5000
  volumes:
    - ./data:/tmp/registry-dev

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
��������
```
  docker-compose up -d
```

## ����
1. ��¼registry
```
docker login -u admin -p admin docker.rd.mt
```
 ��ʾLogin Succeeded˵�����óɹ�

2. ���;��� 
```
docker tag helloworld docker.rd.mt/helloworld
docker push docker.rd.mt/helloworld
```







    
 






