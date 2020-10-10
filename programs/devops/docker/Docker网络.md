[TOC]

# linux系统网桥管理工具brctl 
```
$ brtcl show

bridge name     bridge id               STP enabled     interfaces
docker0         8000.02422f5bcbdf       no
```

# 单机模式
Docker自身的4种网络工作方式，和一些自定义网络模式。安装Docker时，它会自动创建三个网络
+ Host NetWork （主机网络）：容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。
+ None NetWork （无网络）：该模式关闭了容器的网络功能。
+ Bridge NetWork（桥接网络）：此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过docker0网桥以及Iptables nat表配置与宿主机通信。
## 默认网络
当你安装Docker时，它会自动创建三个网络：
```
$ docker network ls
350fff4076f2        bridge              bridge              local
9d41fd71b886        host                host                local
df1350d052e2        none                null                local
```
## 指定网络命令
```
$ docker run --network=<NETWORK>
$ docker run --net=[host|none|bridge]
$ docker run --net=[container]
```
## bridge（网桥）【network namespace隔离，隐藏】
首先，默认情况下docker运行容器时，宿主机会创建一个bridge网桥，是一个名叫docker0的虚拟网桥 ，默认docker0的ip为172.17.0.1，网桥再给容器分配虚拟子网ip，并且以网桥ip作为网关。在不指定网络的情况下，容器之间的通信都是通过bridge网桥进行通信。然后网桥在与宿主机镜像进行ip转换，端口映射等通信。

其实这种bridge网桥与容器，与宿主机之间的通信，有过网络方面经验的同学，看一下下面一张图，就应该可以轻松的了解这种通信的原理了，有点类似与三层路由交换。
![未命名文件 (2)](https://gitee.com/caijingquan/imagebed/raw/master/1602321524_20200102112946027_2077198065.png)
使用bridge最绑定端口
```
$ docker run --net=host -d -p8080:8080 springboot
$ docker exec container ip a

docker0上会生成一个ip
```
## host（宿主机）【同一个network namespace】
如果容器指定网络模式为host，容器不会有自己的network namespace，而是和宿主机共用一个network网络及ip，容器不会有虚拟出自己的网卡、ip等，当然除了网络通信这一块和宿主机绑定了，其余的容器内容还是和宿主机安全隔离了。这种在做容器迁移时，很不方便，不推荐使用。原理图示如下
![未命名文件 (3)](https://gitee.com/caijingquan/imagebed/raw/master/1602321525_20200102113447047_1628707199.png) 
ip与host相同，指定host可以不绑定ip了
```
$ docker run --net=host -d -p8080:8080 springboot
$ docker exec container ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 54:a0:50:d6:28:ad brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.123/24 brd 192.168.2.255 scope global noprefixroute dynamic eno1
       valid_lft 34075sec preferred_lft 34075sec
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```
## none【namespace完全隐藏】
容器指定网络模式-net为none时，docker容器拥有自己的network namespace，但是所有网络配置都得自行配置，如ip、网卡等，这种方式很麻烦，不推荐使用，原理图示如下
![未命名文件 (4)](https://gitee.com/caijingquan/imagebed/raw/master/1602321526_20200102113628578_2078458129.png)
# Container
在理解了host模式后，这个模式也就好理解了。这个模式指定新创建的容器和已经存在的一个容器共享一个Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过lo网卡设备通信。



# 练习
```
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02425f21c208       no

# 运行容器;
$ docker run --name=nginx_bridge --net=bridge -p 80:80 -d nginx
9582dbec7981085ab1f159edcc4bf35e2ee8d5a03984d214bce32a30eab4921a

# 查看容器;
$ docker ps
CONTAINER ID        IMAGE          COMMAND                  CREATED             STATUS              PORTS                NAMES
9582dbec7981        nginx          "nginx -g 'daemon ..."   3 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp   nginx_bridge

# 查看容器网络;
$ docker inspect 9582dbec7981
"Networks": {
    "bridge": {
        "IPAMConfig": null,
        "Links": null,
        "Aliases": null,
        "NetworkID": "9e017f5d4724039f24acc8aec634c8d2af3a9024f67585fce0a0d2b3cb470059",
        "EndpointID": "81b94c1b57de26f9c6690942cd78689041d6c27a564e079d7b1f603ecc104b3b",
        "Gateway": "172.17.0.1",
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "MacAddress": "02:42:ac:11:00:02"
    }
}

# docker查看network namespace
docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "9e017f5d4724039f24acc8aec634c8d2af3a9024f67585fce0a0d2b3cb470059",
        "Created": "2017-08-09T23:20:28.061678042-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "Containers": {
            "9582dbec7981085ab1f159edcc4bf35e2ee8d5a03984d214bce32a30eab4921a": {
                "Name": "nginx_bridge",
                "EndpointID": "81b94c1b57de26f9c6690942cd78689041d6c27a564e079d7b1f603ecc104b3b",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

# 多机模式
+ overlay network（覆盖网络）