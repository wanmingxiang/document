# Linux修改静态IP

​		vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

```sh
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=0ff614b4-6c6d-39c6-be39-b697301d64fe
DEVICE=enp0s3
ONBOOT=yes
IPADDR=192.168.1.106
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1
HWADDR=08:00:27:2e:16:32
```

​		1, BOOTPROTO="dhcp"改成BOOTPROTO="static"

​		2, ONBOOT=TRUE

​	    3，如果配置文件是从别的机器拷贝的， HWADDR 可能跟本机不匹配，使用ip link 命令：

```sh
[root@localhost ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:2e:16:32 brd ff:ff:ff:ff:ff:ff

```

​		08:00:27:2e:16:32即为本地MAC；

​		4， UUID 如何获得：

```sh
[root@localhost ~]#  nmcli connection show
NAME    UUID                                  TYPE      DEVICE
enp0s3  0ff614b4-6c6d-39c6-be39-b697301d64fe  ethernet  enp0s3

```



----

​	vi /etc/udev/rules.d/70-persistent-net.rules

​	NAME=“eth0”修改为NAME=“enp0s3”, reboot;





