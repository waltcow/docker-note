# Experimental Docker Libnetwork DHCP Driver
DHCP driver 让用户能够将在Docker中管理P地址时与现有网卡所在的子网的DHCP服务器中进行动态地址分配的IPAM策略相集成。
DHCP使用户能够以有组织的方式分配地址，通过将来自容器eth0以太网接口的唯一MAC地址与由DHCP配置中定义的DHCP池确定的IP地址相关联，防止重叠的IP地址分配。

此驱动程序仅提供DHCP客户端功能。 它不包括DHCP服务器。 默认驱动程序提供单主机IPAM或分布式多主机协调IPAM请参见libnetwork覆盖驱动程序。

### Getting Started

- 现在最新编译的集成DHCP特性的docker 编译后的文件 [docker-1.13.0-dev.zip](http://git.mailtech.cn/zfmai/libnetwork/blob/master/1.13.0-dev.zip)

- libnetwork 中实现dhcp driver的 [分支](http://git.mailtech.cn/zfmai/libnetwork).

- DHCP 协议的标准参考文档 [DHCP RFC 2131](https://www.ietf.org/rfc/rfc2131.txt)

### DHCP Driver Client Usages

默认情况下，DHCP客户端驱动程序将通过使用DHCP DISCOVER广播自动探测附加到“eth0”的网络。
如果广播域上有DHCP服务器或由中继/帮助代理处理，现有DHCP服务器将回复有关现有可用网络的详细信息。
然后，驱动程序将为使用发现的信息的用户创建Docker IPAM池和网络。

**Note:** 如果docker是在虚拟机上面运行的, 那么你的VM上的网卡需要开启**混杂模式 ** `ifconfig eth0 promisc`
 
```
# Create a network using eth0 as the parent and interface to discover what subnet to use for the network eth0 is attached
docker network create -d macvlan \
  --ipam-driver=dhcp \
  -o parent=eth0 \
  --ipam-opt dhcp_interface=eth0 mcv0

# When containers are created, DHCP requests will assign an IP addresses to each container
docker run --net=mcv0 -itd alpine /bin/sh
docker run --net=mcv0 -itd alpine /bin/sh

# When the containers are destroyed, the DHCP client driver sends a DHCP release and returns the IP address back to the DHCP servers address pool
docker rm -f `docker ps -qa`
docker network rm mcv0
```

