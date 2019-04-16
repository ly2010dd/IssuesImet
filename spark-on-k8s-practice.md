# spark on k8s tke practice
> 参考：
> - 前人在TKE上的实践
https://docs.qq.com/doc/DSGNXQ1Zjd2FDVW9G
> - hdfs on k8s yaml参考
https://github.com/apache-spark-on-k8s/kubernetes-HDFS

### 环境
```
hdfs的docker镜像：sherrytima/hadoop:xxx
spark的docker镜像：sherrytima/spark2.4.0:xxx
TKE集群：
master：129.211.43.170 / 172.17.0.8 内存型M4 8 核 64 GB 10 Mbps 高性能云硬盘50GB SSD云硬盘100GB
worker-0-5：172.81.211.160 / 172.17.0.5 大数据型D2 32 核 128 GB 1 Mbps 高性能云硬盘50GB 本地SATA硬盘11176GB*4 SSD云硬盘100GB
worker-0-6：129.211.87.73 / 172.17.0.6 大数据型D2 32 核 128 GB 1 Mbps 高性能云硬盘50GB 本地SATA硬盘11176GB*4 SSD云硬盘100GB
worker-0-7：172.81.247.88 / 172.17.0.7 大数据型D2 32 核 128 GB 1 Mbps 高性能云硬盘50GB 本地SATA硬盘11176GB*4 SSD云硬盘100GB
worker-0-9：212.64.86.15 / 172.17.0.9 大数据型D2 32 核 128 GB 1 Mbps 高性能云硬盘50GB 本地SATA硬盘11176GB*4 SSD云硬盘100GB
worker-0-10：212.64.64.102 / 172.17.0.10 大数据型D2 32 核 128 GB 1 Mbps 高性能云硬盘50GB 本地SATA硬盘11176GB*4 SSD云硬盘100GB
worker-0-11：129.211.46.64 / 172.17.0.11 大数据型D2 32 核 128 GB 1 Mbps 高性能云硬盘50GB 本地SATA硬盘11176GB*4 SSD云硬盘100GB
```

### 一、搭建hdfs on k8s
1. 制作docker镜像sherrytima/hadoop:xxx

```
docker login 
docker pull kubeguide/hadoop:latest
或docker pull prographerj/centos7-hadoop
docker image ls
docker push sherrytima/hadoop:0.2
```

2. 声明k8s的yaml文件使k8s可以根据yaml编排
    
    hadoop_configmap.yaml
    > 类型是ConfigMap，作为其他yaml文件的配置参数来源
    
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: k8s-hadoop-conf
  namespace: default
data:
  HDFS_MASTER_SERVICE_HOST: "172.17.0.8"
  HDFS_MASTER_SERVICE_PORT: "32001"
  HDFS_DEFAULTFS_PORT: "9000"
  HDFS_NAMENODE_DIR: "/data/nn"
  HDFS_DATA_DIR: "/data/mnt-c/dn,/data/mnt-d/dn,/data/mnt-e/dn,/data/mnt-f/dn"
```
    
namenode.yaml
> 类型是Pod，pod是K8s中调度的最小单位，一个pod中可以放一个或者多个container

```
---
apiVersion: v1
kind: Pod
metadata:
    name: k8s-hadoop-master
    labels:
        app: k8s-hadoop-master
spec:
    containers:
        - name: k8s-hadoop-master
          image: sherrytima/hadoop:0.4
          imagePullPolicy: IfNotPresent
          command: [ "/bin/bash", "-ce", "tail -f /dev/null" ]
          ports:
            - containerPort: 9000
            - containerPort: 50070
          env:
            - name: HADOOP_NODE_TYPE
              value: namenode
            - name: HDFS_SERVICE_HOST
              valueFrom:
                    configMapKeyRef:
                        name: k8s-hadoop-conf
                        key: HDFS_MASTER_SERVICE_HOST
            - name: HDFS_SERVICE_PORT
              valueFrom:
                    configMapKeyRef:
                        name: k8s-hadoop-conf
                        key: HDFS_MASTER_SERVICE_PORT
            - name: HDFS_NAME_DIR
              valueFrom:
                    configMapKeyRef:
                        name: k8s-hadoop-conf
                        key: HDFS_NAMENODE_DIR
            - name: HDFS_DATA_DIR
              valueFrom:
                    configMapKeyRef:
                        name: k8s-hadoop-conf
                        key: HDFS_DATA_DIR
            - name: HDFS_DEFAULTFS_PORT
              valueFrom:
                    configMapKeyRef:
                        name: k8s-hadoop-conf
                        key: HDFS_DEFAULTFS_PORT
          volumeMounts:
            - mountPath: /data/nn
              name: namenode-dir
    volumes:
    - name: namenode-dir
      hostPath:
        path: /var/lib/docker/hadoop

    nodeSelector:
      node: k8s-hadoop-master
    hostNetwork: true
    restartPolicy: Always
```
    
datanode_daemon_1.yaml
> 类型是DaemonSet，daemonset使每个node上都只运行一个pod，适合存储类型的工作如数据库、写日志等

``` 
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
    name: k8s-hadoop-datanode-1
    labels:
        app: k8s-hadoop-datanode-1
spec:
  selector:
    matchLabels:
      name: k8s-hadoop-datanode-1
  template:
    metadata:
      labels:
        name: k8s-hadoop-datanode-1
    spec:
      containers:
        - name: k8s-hadoop-datanode-1
          image: sherrytima/hadoop:0.4
          imagePullPolicy: IfNotPresent
          command: [ "/bin/bash", "-ce", "./root/start-node.sh" ]
          resources:
            limits:
              memory: 8192Mi
              cpu: 8000m
            requests:
              cpu: 2000m
              memory: 2048Mi
          env:
            - name: HADOOP_NODE_TYPE
              value: datanode
            - name: HDFS_SERVICE_HOST
              valueFrom:
                configMapKeyRef:
                  name: k8s-hadoop-conf
                  key: HDFS_MASTER_SERVICE_HOST
            - name: HDFS_SERVICE_PORT
              valueFrom:
                configMapKeyRef:
                  name: k8s-hadoop-conf
                  key: HDFS_MASTER_SERVICE_PORT
            - name: HDFS_DATA_DIR
              valueFrom:
                configMapKeyRef:
                  name: k8s-hadoop-conf
                  key: HDFS_DATA_DIR
            - name: HDFS_NAME_DIR
              valueFrom:
                configMapKeyRef:
                  name: k8s-hadoop-conf
                  key: HDFS_NAMENODE_DIR
            - name: HDFS_DEFAULTFS_PORT
              valueFrom:
                    configMapKeyRef:
                        name: k8s-hadoop-conf
                        key: HDFS_DEFAULTFS_PORT
          volumeMounts:
            - name: data1
              mountPath: /data/mnt-c
            - name: data2
              mountPath: /data/mnt-d
            - name: data3 mountPath: /data/mnt-e
            - name: data4
              mountPath: /data/mnt-f
      volumes:
      - name: data1
        hostPath:
          path: /data/mnt-c
      - name: data2
        hostPath:
          path: /data/mnt-d
      - name: data3
        hostPath:
          path: /data/mnt-e
      - name: data4
        hostPath:
          path: /data/mnt-f
      nodeSelector:
        node: k8s-hadoop-datanode-1
      hostNetwork: true
      hostPID: true
      restartPolicy: Always
```
    
datanode_daemon_x.yaml
```
同上，只需把对应位置的数据改为x，此环境起了6个datanode的pod，所以需要写6个yaml文件
```

service.yaml
> 类型是Service，由于pod之间是内网通信，需要一个service作为代理与外部通信

```
apiVersion: v1
kind: Service
metadata:
        name: k8s-hadoop-master
spec:
        type: NodePort
        selector:
                app: k8s-hadoop-master
        ports:
            - name: rpc
              port: 9000
              targetPort: 9000
              nodePort: 32001
            - name: http
              port: 50070
              targetPort: 50070
              nodePort: 32007
```

3. 修改每台（master/worker1/worker2/worker3）机器的/etc/hosts

运行
```
hostname VM-0-9-centos
hostname -f
```
修改/etc/hostname
```
VM-0-8-centos
```
修改/etc/hosts
```
127.0.0.1 localhost
172.17.0.5 VM-0-5-centos
172.17.0.6 VM-0-6-centos
172.17.0.7 VM-0-7-centos
172.17.0.8 VM-0-8-centos
172.17.0.9 VM-0-9-centos
172.17.0.10 VM-0-10-centos
172.17.0.11 VM-0-11-centos
```

4. 设置node lables
    
    > 设置node labels是为使对应的yaml文件找到相应的label    

```
kubectl get nodes --show-labels
kubectl label nodes 172.17.0.5 node=k8s-hadoop-datanode-1
kubectl label nodes 172.17.0.6 node=k8s-hadoop-datanode-2
kubectl label nodes 172.17.0.7 node=k8s-hadoop-datanode-3
kubectl label nodes 172.17.0.5 node=k8s-hadoop-datanode-4
kubectl label nodes 172.17.0.6 node=k8s-hadoop-datanode-5
kubectl label nodes 172.17.0.7 node=k8s-hadoop-datanode-6
kubectl label nodes 172.17.0.8 node=k8s-hadoop-master
# 修改 
kubectl label nodes 172.17.0.7 node=k8s-hadoop-datanode-3 --overwrite=true
# 删除 
kubectl label nodes 172.17.0.7 node-
```

5. 使yaml文件生效
   
> 注意这里是有顺序的，先configmap->service->namenode->datanode 

```
kubectl create -f hadoop_configmap.yaml
kubectl create -f service.yaml
kubectl create -f namenode.yaml
kubectl create -f datanode_daemon_1.yaml
kubectl create -f datanode_daemon_2.yaml
kubectl create -f datanode_daemon_3.yaml
kubectl create -f datanode_daemon_4.yaml
kubectl create -f datanode_daemon_5.yaml
kubectl create -f datanode_daemon_6.yaml
```

6. 查看已启动的pod状态命令
    
```
kubectl get pods
kubectl get daemonsets
kubectl get services
kubectl get configmaps
kubectl describe pod xxx
```

7. 进入pod命令，执行jps查看每个pod里namenode和datanode是否都运行着
    
```
kubectl exec -it k8s-hadoop-master /bin/bash
kubectl exec -it k8s-hadoop-datanode-1-vjszx /bin/bash
jps
```

8. 验证hdfs是否运行正确
    
进入namenode pod
```
kubectl exec -it k8s-hadoop-master /bin/bash
```

初始化磁盘并挂载（只有第一次需要）
```
./init.sh
```

启动namenode
```
./start-node.sh
jps
```

启动datanode pod
```
kubectl exec -it datanode-pod-name /bin/bash
./init.sh
./start-node.sh
jps
```

备注：init.sh如下
```
yes |mkfs.ext4 /dev/vdc
yes |mkfs.ext4 /dev/vdd
yes |mkfs.ext4 /dev/vde
yes |mkfs.ext4 /dev/vdf

mkdir -p /data/mnt-c
mkdir -p /data/mnt-d
mkdir -p /data/mnt-e
mkdir -p /data/mnt-f

mount /dev/vdc /data/mnt-c
mount /dev/vdd /data/mnt-d
mount /dev/vde /data/mnt-e
mount /dev/vdf /data/mnt-f
```

备注：start-node.sh如下
```
#!/usr/bin/env bash
sed -i "s/@HDFS_SERVICE_HOST@/$HDFS_SERVICE_HOST/g" $HADOOP_HOME/etc/hadoop/core-site.xml
sed -i "s/@HDFS_DEFAULTFS_PORT@/$HDFS_DEFAULTFS_PORT/g" $HADOOP_HOME/etc/hadoop/core-site.xml
sed -i "s|@HDFS_DATA_DIR@|$HDFS_DATA_DIR|g" $HADOOP_HOME/etc/hadoop/hdfs-site.xml
sed -i "s|@HDFS_NAME_DIR@|$HDFS_NAME_DIR|g" $HADOOP_HOME/etc/hadoop/hdfs-site.xml

HADOOP_NODE="${HADOOP_NODE_TYPE}"

if [ $HADOOP_NODE = "datanode" ]; then
        echo "Start DataNode ..."
        hdfs datanode  -regular
else
        if [  $HADOOP_NODE = "namenode" ]; then
                echo "Start NameNode ..."
                if [ ! -f $HDFS_NAME_DIR/current/VERSION ]; then
                        hdfs namenode -format -force
                fi
                $HADOOP_HOME/sbin/hadoop-daemon.sh start namenode
        fi
fi
```

验证hdfs
```
#进入namenode的pod里查看report
hdfs dfsadmin -report
```

9. 遇见的坑与解决办法
> 参考：k8s上运行hdfs的问题
https://docs.qq.com/doc/DSmp0VXFheFZCdmNi?opendocxfrom=admin


### 二、spark on k8s跑sparkpi任务

> 参考：spark官网 spark on k8s
https://spark.apache.org/docs/latest/running-on-kubernetes.html

1. 查看k8s master通信的ip和port
```
[root@VM_0_8_centos ~]# kubectl cluster-info
Kubernetes master is running at https://169.254.128.15:60002
KubeDNS is running at https://169.254.128.15:60002/api/v1/namespaces/kube-system/services/kube-dns:dns-tcp/proxy
```

2. 从spark镜像加载jar包跑sparkpi任务
```
bin/spark-submit \
--master k8s://https://169.254.128.15:60002 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=5 \
--conf spark.driver.memory=1G \
--conf spark.driver.cores=1 \
--conf spark.executor.memory=4G \
--conf spark.executor.cores=1 \
--conf spark.kubernetes.container.image=sherrytima/spark2.4.0:0.2 \
local:///opt/spark/examples/jars/spark-examples_2.11-2.4.0.jar

# 注：
# 1. 这里的master中的ip和port是通过kubectl cluster-info获取
# 2. deploy-mode使用cluster模式执行。cluster模式是将spark-driver启在集群内部，方便driver与executors通信，但driver与spark-client通信会有一定延迟。client模式是将spark-driver启在与spark-client相同的机器上，方便二者通信，但driver与集群内的executors通信会有延迟。
# 3. local://表示jar包从spark镜像中加载
# 4. 如果执行失败，则通过kubectl logs spark-terasort-100g-12-1554365466120-driver命令查看log定位问题
# 5. 如果执行失败log显示缺少libnss3.so，解决方法参考遇见的坑与解决办法
```

3. 从hdfs镜像加载jar包跑sparkpi任务
进入namenode的pod
```
kubectl exec -it k8s-hadoop-master /bin/bash
```
将jar包传到hdfs的/user路径下
```
hdfs dfs -put -f spark-examples_2.11-2.4.0.jar /user
```
提交sparkpi任务
```
bin/spark-submit \
--master k8s://https://169.254.128.15:60002 \
--deploy-mode cluster \
--name spark-pi \
--class org.apache.spark.examples.SparkPi \
--conf spark.executor.instances=5 \
--conf spark.driver.memory=1G \
--conf spark.driver.cores=1 \
--conf spark.executor.memory=4G \
--conf spark.executor.cores=1 \
--conf spark.kubernetes.container.image=sherrytima/spark2.4.0:k8s3 \
hdfs://172.17.0.8:32091/user/spark-examples_2.11-2.4.0.jar

# 注：
# 1. hdfs://后的ip:port是namenode节点core-site.xml里的fs.defaultFS配置的
```

4. 遇见的坑与解决办法
> 参考：k8s上运行spark的问题
https://docs.qq.com/doc/DSnlobndvY3l6T1V4?opendocxfrom=admin

### 三、spark on k8s跑spark-terasort 100g排序

> 参考：spark-terasort官网
https://github.com/ehiggs/spark-terasort

1. 编译
```
mvn install -Dspark.version=2.4.0

#注：编译后的jar包是target目录中的spark-terasort-1.1-SNAPSHOT-jar-with-dependencies.jar
```

2. 进入namenode的pod
```
kubectl exec -it k8s-hadoop-master /bin/bash
```

3. 将jar包传到hdfs的/user路径下
```
hdfs dfs -put -f spark-terasort-1.1-SNAPSHOT-jar-with-dependencies.jar /user
```

4. Generate data
```
bin/spark-submit \
--master k8s://https://169.254.128.15:60002 \
--deploy-mode cluster \
--name spark-teragen-100g \
--class com.github.ehiggs.spark.terasort.TeraGen \
--conf spark.executor.instances=5 \
--conf spark.driver.memory=50G \
--conf spark.driver.cores=6 \
--conf spark.executor.memory=100G \
--conf spark.executor.cores=24 \
--conf spark.kubernetes.container.image=sherrytima/spark2.4.0:k8s5 \
hdfs://172.17.0.8:32091/user/spark-terasort-1.1-SNAPSHOT-jar-with-dependencies.jar 1g hdfs://172.17.0.8:32091/user/terasort_in

# 注：这里配置的参数driver（50G/6cores）、executor（100G/24cores）是根据当前TKE集群配置进行测试得到的最大使用参数，如果再调大log中将抛异常
# Initial job has not accepted any resources; check your cluster UI to ensure that workers are registered and have sufficient resources
```

5. Sort the data
```
bin/spark-submit \
--master k8s://https://169.254.128.15:60002 \
--deploy-mode cluster 
--name spark-terasort-100g \
--class com.github.ehiggs.spark.terasort.TeraSort \
--conf spark.executor.instances=5 \
--conf spark.driver.memory=50G \
--conf spark.driver.cores=6 \
--conf spark.executor.memory=100G \
--conf spark.executor.cores=24 \
--conf spark.kubernetes.container.image=sherrytima/spark2.4.0:k8s5 \
hdfs://172.17.0.8:32091/user/spark-terasort-1.1-SNAPSHOT-jar-with-dependencies.jar hdfs://172.17.0.8:32091/user/terasort_in hdfs://172.17.0.8:32091/user/terasort_out
```

6. Validate the data
```
bin/spark-submit \
--master k8s://https://169.254.128.15:60002 \
--deploy-mode cluster \
--name spark-teravalidate-100g \
--class com.github.ehiggs.spark.terasort.TeraValidate \
--conf spark.executor.instances=5 \
--conf spark.driver.memory=50G \
--conf spark.driver.cores=6 \
--conf spark.executor.memory=100G \
--conf spark.executor.cores=24 \
--conf spark.kubernetes.container.image=sherrytima/spark2.4.0:k8s5 \
hdfs://172.17.0.8:32091/user/spark-terasort-1.1-SNAPSHOT-jar-with-dependencies.jar hdfs://172.17.0.8:32091/user/terasort_out hdfs://172.17.0.8:32091/user/terasort_validate
```

### 三、spark on k8s跑TPCDS 1t

启动mysql
```
service mysql start
```

mysql对其他ip进行授权密码访问
```
mysql -uroot -p
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
flush privileges;
```

配置hive
```
#进入namenode pod，也可以起在datanode pod中
kubectl exec -it k8s-hadoop-master /bin/bash

#/opt/hive/conf
<?xml version="1.0" ?>
<configuration>
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://172.17.0.8:3306/metastore?createDatabaseIfNotExist=true&amp;useSSL=false</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>Spark888$</value>
  </property>
  <property>
    <name>datanucleus.autoCreateSchema</name>
    <value>false</value>
  </property>
  <property>
    <name>datanucleus.autoCreateTables</name>
    <value>true</value>
  </property>
  <property>
    <name>datanucleus.autoStartMechanism</name>
    <value>SchemaTable</value>
  </property>
  <property>
    <name>datanucleus.fixedDatastore</name>
    <value>true</value>
  </property>
  <property>
    <name>hive.metastore.uris</name>
    <value>thrift://172.17.0.8:9083</value>
  </property>
  <property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
  </property>
  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>hdfs://172.17.0.8:32001/spark-warehouse</value>
  </property>

  <property>
    <name>hive.server2.session.check.interval</name>
    <value>3600000</value>
  </property>
  <property>
    <name>hive.server2.idle.operation.timeout</name>
    <value>86400000</value>
  </property>
  <property>
    <name>hive.server2.idle.session.timeout</name>
    <value>259200000</value>
  </property>

</configuration>
```

启动hive
```
export HIVE_HOME=/opt/hive
nohup ${HIVE_HOME}/bin/hive --service metastore > /dev/null 2>&1 &
jps
```

编译spark tagv2.4.1
```
export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m"
./dev/make-distribution.sh --name v2.4.1 --pip --tgz -Phadoop-2.7 -Phive -Phive-thriftserver -Pyarn -Pkubernetes
参考：https://spark.apache.org/docs/latest/building-spark.html#building-a-runnable-distribution
```

解压spark-bin.tar.gz到/opt/spark
```
ln -s spark-dir spark
```

将需要放入spark docker镜像的东西放到/opt/spark/spark_docker_image_dependencies下，需要需要以下
```
ls /opt/spark/spark_docker_image_dependencies
hive-site.xml  hive-v2_1-dependencies  spark-defaults.conf  tpcds-kit
```

备注：hive-site.xml
```
同配置hive
```

备注：spark-defaults.conf
```
spark.sql.hive.metastore.version 2.1.0
spark.sql.hive.metastore.jars /opt/spark/hive-v2_1-dependencies/*
```

备注：hive-v2_1-dependencies是hive中所有的jars
```
备份放在service/backup
```

备注：tpcds-kit是tpcds依赖的一些工具包，应该放在spark镜像里
```
下载：https://github.com/databricks/tpcds-kit.git
备份放在service/backup
```

在$SPARK_HOME/kubernetes/dockerfiles/spark/Dockerfile中添加
```
COPY conf /opt/spark/conf
COPY spark_docker_image_dependencies/hive-site.xml /opt/spark/conf
COPY spark_docker_image_dependencies/spark-defaults.conf /opt/spark/conf
COPY spark_docker_image_dependencies/hive-v2_1-dependencies /opt/spark/hive-v2_1-dependencies
COPY spark_docker_image_dependencies/tpcds-kit /opt/tpcds-kit
```

制作spark2.4.1镜像
```
/opt/spark/build-spark-image.sh image-fullname
eg: /opt/spark/build-spark-image.sh sherrytima/spark2.4.1:k8s1
```

备注：build-spark-image.sh
```
#!/bin/bash

if [ ! -n "$1" ] ;then
    echo "usage:
    ./build-spark-image.sh docker-image-fullname

    eg: ./build-spark-image.sh sherrytima/spark2.4.1:test 

    Notice: add following infomation into \$SPARK_HOME/kubernetes/dockerfiles/spark/Dockerfile firstly!
        COPY conf /opt/spark/conf
        COPY spark_docker_image_dependencies/hive-site.xml /opt/spark/conf
        COPY spark_docker_image_dependencies/spark-defaults.conf /opt/spark/conf
        COPY spark_docker_image_dependencies/hive-v2_1-dependencies /opt/spark/hive-v2_1-dependencies
        COPY spark_docker_image_dependencies/tpcds-kit /opt/tpcds-kit    
    "
    exit 1;
fi

cp -r /opt/spark/spark_docker_image_dependencies/ /opt/spark/spark
cd /opt/spark/spark
docker build -t $1 -f kubernetes/dockerfiles/spark/Dockerfile .
```

配置master node中的spark-submit时的spark-defaults.conf
```
spark.master  k8s://https://169.254.128.15:60002
spark.submit.deployMode cluster
spark.executor.instances 24
spark.executor.memory 24g
spark.driver.memory 8g
spark.driver.memoryOverhead 2g
spark.executor.memoryOverhead 4g
spark.driver.cores 2
spark.executor.cores 6
#spark.kubernetes.authenticate.driver.serviceAccountName=spark
#spark.kubernetes.container.image=sherrytima/spark2.4.1:k8s1
#spark.eventLog.enabled true
#spark.eventLog.dir hdfs://172.17.0.8:32001/spark-logs
#spark.history.fs.logDirectory hdfs://172.17.0.8:32001/spark-logs
#spark.eventLog.compress true
spark.kubernetes.allocation.batch.size 4
spark.kubernetes.executor.label.role=spark-exec
spark.kubernetes.driver.label.role=spark-driver
spark.kubernetes.node.selector.role=compute
#network
spark.network.timeout 1200s
spark.port.maxRetries 16
spark.rpc.numRetries 10

#sql
spark.sql.shuffle.partitions 1024
spark.sql.autoBroadcastJoinThreshold 33554432

spark.task.cpus 1
spark.task.maxFailures 10

spark.files.fetchTimeout 600s
spark.driver.maxResultSize 4g

#spark.kubernetes.driver.label.SPARK=master

spark.kubernetes.executor.volumes.hostPath.spark-local-dir-1.mount.path=/data/mnt-c
spark.kubernetes.executor.volumes.hostPath.spark-local-dir-1.options.path=/data/mnt-c
#spark.kubernetes.executor.volumes.hostPath.local-dir1.options.type=DirectoryOrCreate

spark.kubernetes.executor.volumes.hostPath.spark-local-dir-2.mount.path=/data/mnt-d
spark.kubernetes.executor.volumes.hostPath.spark-local-dir-2.options.path=/data/mnt-d
#spark.kubernetes.executor.volumes.hostPath.local-dir2.options.type=DirectoryOrCreate

spark.kubernetes.executor.volumes.hostPath.spark-local-dir-3.mount.path=/data/mnt-e
spark.kubernetes.executor.volumes.hostPath.spark-local-dir-3.options.path=/data/mnt-e
#spark.kubernetes.executor.volumes.hostPath.local-dir3.options.type=DirectoryOrCreate

spark.kubernetes.executor.volumes.hostPath.spark-local-dir-4.mount.path=/data/mnt-f
spark.kubernetes.executor.volumes.hostPath.spark-local-dir-4.options.path=/data/mnt-f
#spark.kubernetes.executor.volumes.hostPath.local-dir4.options.type=DirectoryOrCreate

spark.local.dir=/data/mnt-c,/data/mnt-d,/data/mnt-e,/data/mnt-f
#spark.kubernetes.local.dirs.tmpfs=true
```

编写tpcds测试脚本test.scala
```
package pers.liy.tpcds

import com.databricks.spark.sql.perf.tpcds.TPCDSTables
import org.apache.spark.sql.SQLContext
import org.apache.spark.SparkContext
import com.databricks.spark.sql.perf.tpcds.TPCDS

object TpcDsGenData{
  def main(args:Array[String]){
    // Set:
    val sc = new org.apache.spark.SparkContext()
    val sqlContext = new org.apache.spark.sql.SQLContext(sc)
    //val rootDir = "hdfs://172.17.0.8:32001/tpcds/"
    //val scaleFactor = "10"
    //val format = "orc"
    val args = sc.getConf.get("spark.driver.args").split("\\s+")
    val rootDir = args(0)
    val scaleFactor = args(1)
    val format = args(2)
    val resultLocation = rootDir + "/" + format + "/" + scaleFactor + "/" + "result"
    val databaseName = "tpcds_" + format + "_" + scaleFactor
    val genDataLocation = rootDir + "/" + format + "/" + scaleFactor + "/" + "data"

    // Run:
    val tables = new TPCDSTables(sqlContext,
        dsdgenDir = "/opt/tpcds-kit/tools", // location of dsdgen
        scaleFactor = scaleFactor,
        useDoubleForDecimal = false, // true to replace DecimalType with DoubleType
        useStringForDate = false) // true to replace DateType with StringType


    tables.genData(
        location = genDataLocation,
        format = format,
        overwrite = true, // overwrite the data that is already there
        partitionTables = true, // create the partitioned fact tables
        clusterByPartitionColumns = true, // shuffle to get partitions coalesced into single files.
        filterOutNullPartitionValues = false, // true to filter out the partition with NULL key value
        tableFilter = "", // "" means generate all tables
        numPartitions = 100) // how many dsdgen partitions to run - number of input tasks.


    // Create the specified database
    sqlContext.sql(s"create database $databaseName")
    // Create metastore tables in a specified database for your data.
    // Once tables are created, the current database will be switched to the specified database.
    tables.createExternalTables(rootDir, format, databaseName, overwrite = true, discoverPartitions = true)
    // Or, if you want to create temporary tables
    // tables.createTemporaryTables(location, format)

    // For CBO only, gather statistics on all columns:
    tables.analyzeTables(databaseName, analyzeColumns = true)

    val tpcds = new TPCDS(sqlContext = sqlContext)
    // Set:
    val iterations = 1 // how many iterations of queries to run.
    val queries = tpcds.tpcds2_4Queries // queries to run.
    val timeout = 24*60*60 // timeout, in seconds.

    // Run:
    sqlContext.sql(s"use $databaseName")
    val experiment = tpcds.runExperiment(
      queries,
      iterations = iterations,
      resultLocation = resultLocation,
      forkThread = true)
    experiment.waitForFinish(timeout)
    }
}

```

将test.scala用sbt打成jar包（spark-submit只支持jar或app）
```
#sbt工程结构
tpcds-liy
├── build.sbt
├── lib
│   ├── com.typesafe.scala-logging_scala-logging-api_2.11-2.1.2.jar
│   ├── com.typesafe.scala-logging_scala-logging-slf4j_2.11-2.1.2.jar
│   ├── org.scala-lang_scala-reflect-2.11.0.jar
│   ├── org.slf4j_slf4j-api-1.7.7.jar
│   └── spark-sql-perf_2.11-0.5.1-SNAPSHOT.jar
├── project
│   └── build.properties
└── src
    └── main
        └── scala
            └── test.scala

#参考：https://console.bluemix.net/docs/services/AnalyticsforApacheSpark/spark_app_example.html
```

备注：build.sbt
```
name := "TPCDS"

version := "1.0"

scalaVersion := "2.11.12"

libraryDependencies += "org.apache.spark" %% "spark-core" % "2.4.1"

libraryDependencies += "org.apache.spark" %% "spark-sql" % "2.4.1"


resolvers += "Akka Repository" at "http://repo.akka.io/releases/"

```

备注：project/build.properties
```
sbt.version=1.2.6

```

sbt打jar包命令
```
sbt package
```

上传tpcds运行需要的jar包到hdfs上run-lib下
```
hdfs dfs -ls /run-lib

-rw-r--r--   3 root supergroup       7694 2019-04-12 12:45 /run-lib/com.typesafe.scala-logging_scala-logging-api_2.11-2.1.2.jar
-rw-r--r--   3 root supergroup      24064 2019-04-12 12:45 /run-lib/com.typesafe.scala-logging_scala-logging-slf4j_2.11-2.1.2.jar
-rw-r--r--   3 root supergroup     933503 2019-04-14 07:45 /run-lib/spark-sql-perf_2.11-0.5.1-SNAPSHOT.jar
```

上传tpcds_2.11-1.0.jar到hdfs上tpcds下
```
hdfs dfs -ls /tpcds

-rw-r--r--   3 root supergroup       3391 2019-04-14 09:09 /tpcds/tpcds_2.11-1.0.jar
```

运行tpcds 1t测试
```
bin/spark-submit  --name tpcds-1t  --jars hdfs://172.17.0.8:32001/run-lib/*  --class pers.liy.tpcds.TpcDsGenData   --conf spark.kubernetes.container.image=sherrytima/spark2.4.1:k8s6  --conf spark.driver.args="hdfs://172.17.0.8:32001/run-tpcds/ 1000 orc"  hdfs://172.17.0.8:32001/tpcds/tpcds_2.11-1.0.jar
```
