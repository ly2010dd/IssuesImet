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

# Hadoop分布式模式

> 参考：
> - Hadoop 2.7 cluster installation and configuration on RHEL7/CentOS7 http://fibrevillage.com/storage/617-hadoop-2-7-cluster-installation-and-configuration-on-rhel7-centos7

### 每台机器创建hadoop账户
```
useradd hadoop
passwd hadoop
```

### 每台机器修改host
```
vim /etc/hosts
192.168.207.111 master
192.168.207.112 slave1
192.168.207.113 slave2
```

### 配置相互免密登录
```
#每台执行
su - hadoop
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@master
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@slave1
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@slave2
exit
```

### 解压tar包在master上
```
mkdir -p /opt/hadoop
chown hadoop:hadoop /opt/hadoop
su - hadoop
cd /opt/hadoop
上传hadoop的tar包
tar xzf hadoop-2.7.3.tar.gz
ln -s  /opt/hadoop/hadoop-2.7.6/ /opt/hadoop/hadoop
```

### 设置环境变量
```
export HADOOP_HOME=/opt/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

source ~/.bashrc
```
```
修改$HADOOP_HOME/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_171
export HADOOP_OPTS=-Djava.net.preferIPv4Stack=true
```

### 配置文件配置
> 参考：
> - CentOS7.5搭建Hadoop2.7.6完全分布式集群
http://www.cnblogs.com/frankdeng/p/9047698.html

集群规划部署

节点名称 | NN1 | NN2  | DN | RM | NM
---|---|---|---|---|---
master | NameNode |     | DN |    | NM
slave1 |          | NN2 | DN | RM | NM
slave2 |          |     | DN |    | NM


core-site.xml
```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/hadoop/tmp</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.groups</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.root.hosts</name>
                <value>*</value>
        </property>
</configuration>
```

hdfs-site.xml
```
<configuration>
    <!-- 设置dfs副本数，不设置默认是3个   -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <!-- 设置secondname的端口   -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>slave1:50090</value>
    </property>

</configuration>
```

slaves
```
master
slave1
slave2
```

mapred-site.xml
```
<configuration>
    <!-- 指定mr运行在yarn上 -->
    <property>
     <name>mapreduce.framework.name</name>
     <value>yarn</value>
    </property>
</configuration>
```

yarn-site.xml
```
<configuration>
     <!-- Site specific YARN configuration properties -->
     <!-- reducer获取数据的方式 -->
     <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
     </property>
     <!-- 指定YARN的ResourceManager的地址 -->
     <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>slave1</value>
     </property>
</configuration>
```

### 拷贝到slave1和slave2
```
# to slave1
scp -r hadoop-2.7.6/ admin@node22:`pwd`scp -r hadoop-2.7.6/ hadoop@slave1:`pwd`
ln -s  /opt/hadoop/hadoop-2.7.6/ /opt/hadoop/hadoop
# to slave2
scp -r hadoop-2.7.6/ admin@node22:`pwd`scp -r hadoop-2.7.6/ hadoop@slave2:`pwd`
ln -s  /opt/hadoop/hadoop-2.7.6/ /opt/hadoop/hadoop
```

### 启动集群
```
hdfs namenode -format　
start-dfs.sh #需要在slave1上执行，因为yarn-site.xml上配置的是在slave1上启动
start-yarn.sh
jps
```
