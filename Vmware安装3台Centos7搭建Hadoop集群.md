# 集群初始环境搭建
> 参考：
> - 通过VMware搭建分布式集群基础环境 https://blog.csdn.net/cndmss/article/details/80149952 
> - 在vmware上安装centos7以及网络配置
https://blog.csdn.net/didi8206050/article/details/51872682

### 安装Centos7
```
40G，不拆分（ntfs）拆分（fat32），2G，1核，网卡nat模式共享主机网卡
```
### 激活网卡
- centos7中查看网卡
```
ip addr
```
- 激活网卡
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
改ONBOOT=yes
service network restart
```
> 此时ping 网关可以ping通，ping百度也可以ping通了

### 配置静态IP
- Vmware中->编辑->虚拟网络编辑器 查看nat网关ip和子网掩码
- 配置网卡
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
改
BOOTPROTO=static
增
IPADDR=192.168.207.111（随意）
NETMASK=255.255.255.0
GATEWAY=192.168.207.2
service network restart
```

### 配置域名解析服务器
```
vi /etc/resolv.conf
增
nameserver 192.168.207.2（网关）
nameserver 8.8.8.8
nameserver 8.8.4.4
```

### 关闭防火墙
```
systemctl list-unit-files|grep enabled
systemctl stop firewalld.service
```

### 安装JDK
```
略
```

### 克隆虚拟机
- 关机
- 拍摄快照
- 克隆（master、slave1、slave2）

### 改静态IP
- 修改slave1和slave2的静态ip
- 修改三台机器的hostname（master、slave1、slave2）
```
vim /etc/hostname
```
