# Hadoop相关问题

> **作者：liy**


## start-dfs.sh启动HDFS发现datanode没有启动
- 解决方法：
进入/tmp/hadoop-root/dfs/目录，分别打开name、data的current文件夹里的VERSION，可以看到clusterID项正如日志里记录的一样，确实不一致，修改datanode里VERSION文件的clusterID 与namenode里的一致，再重新启动dfs（执行start-dfs.sh）再执行jps命令可以看到datanode已正常启动。