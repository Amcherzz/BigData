---
title: Hadoop环境搭建
date: 2020-04-28 21:34:25
tags:
	- 环境搭建
categories: Hadoop
---

以下为自己搭建hadoop集群学习使用

## 防火墙设置

关闭防火墙，关闭防火墙开机自启动
```
systemctl stop firewalld
systemctl disable firewalld
```

## 创建用户、修改密码

```
useradd usename
passwd password
```
<!-- more -->
## 配置用户root权限，方便后续使用

```
vim /etc/sudoers
```

```
root    ALL=(ALL)     ALL
username   ALL=(ALL)     NOPASSWD:ALL
```

## 设置主机名及主机地址映射文件

注意hosts文件配置，其他节点配公网ip，自己配内网ip

```
vim /etc/hostname
vim /etc/hosts
```

## 安装JDK8

卸载原有的
```
rpm -qa | grep -i java | xargs -n1 sudo rpm -e --nodeps
```
解压到module目录下
```
tar -zxvf jdk-8u212-linux-x64.tar.gz -C /opt/module/
```
配置环境变量
```
sudo vim /etc/profile.d/my_env.sh
```
```
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin
```

```
source /etc/profile
```
查看是否安装成功
```
java -version
```

## 安装Hadoop

解压
```
tar -zxvf hadoop-3.1.3.tar.gz -C /opt/module/
```

配置环境变量
```
sudo vim /etc/profile.d/my_env.sh
```
```
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```
```
source /etc/profile
```
测试是否安装成功
```
hadoop version
```

## 配置

- 本地测试

  ```
  mkdir wcinput
  cd wcinput
  vim word.txt
  
  world china
  nice nice
  good java scala python
  python
  
  # 注意root和一般用户切换，并刷新环境变量
  
  hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount wcinput wcoutput
  ```

- 各个节点同步数据

  ```
  #!/bin/bash
  #1. 判断参数个数
  if [ $# -lt 1 ]
  then
    echo Not Enough Arguement!
    exit;
  fi
  #2. 遍历集群所有机器
  for host in hadoop1 hadoop2 hadoop3
  do
    echo ====================  $host  ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
      #4. 判断文件是否存在
      if [ -e $file ]
      then
        #5. 获取父目录
        pdir=$(cd -P $(dirname $file); pwd)
        #6. 获取当前文件的名称
        fname=$(basename $file)
        ssh $host "mkdir -p $pdir"
        rsync -av $pdir/$fname $host:$pdir
      else
        echo $file does not exists!
      fi
    done
  done
  ```

  ```
  chmod +x xsync
  sudo cp xsync /bin/
  ```

- ssh免密登录配置

  ```
  ssh-keygen -t rsa
  ssh-copy-id hadoop1
  ssh-copy-id hadoop2
  ssh-copy-id hadoop3
  ```

- 文件配置

  ```
  cd $HADOOP_HOME/etc/hadoop
  vim core-site.xml
  ```

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
  	<!-- 指定NameNode的地址 -->
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://hadoop1:8020</value>
  </property>
  <!-- 指定hadoop数据的存储目录 -->
      <property>
          <name>hadoop.tmp.dir</name>
          <value>/opt/module/hadoop-3.1.3/data</value>
  </property>
  
  <!-- 配置HDFS网页登录使用的静态用户为llc -->
      <property>
          <name>hadoop.http.staticuser.user</name>
          <value>llc</value>
  </property>
  
  <!-- 配置该llc(superUser)允许通过代理访问的主机节点 -->
      <property>
          <name>hadoop.proxyuser.llc.hosts</name>
          <value>*</value>
  </property>
  <!-- 配置该llc(superUser)允许通过代理用户所属组 -->
      <property>
          <name>hadoop.proxyuser.llc.groups</name>
          <value>*</value>
  </property>
  <!-- 配置该llc(superUser)允许通过代理的用户-->
      <property>
          <name>hadoop.proxyuser.llc.groups</name>
          <value>*</value>
  </property>
  </configuration>
  ```

  hdfs-site.xml

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
  	<!-- nn web端访问地址-->
  	<property>
          <name>dfs.namenode.http-address</name>
          <value>hadoop1:9870</value>
      </property>
  	<!-- 2nn web端访问地址-->
      <property>
          <name>dfs.namenode.secondary.http-address</name>
          <value>hadoop3:9868</value>
      </property>
  </configuration>
  ```

  yarn-site.xml

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
  	<!-- 指定MR走shuffle -->
      <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
  </property>
  <!-- 指定ResourceManager的地址-->
      <property>
          <name>yarn.resourcemanager.hostname</name>
          <value>hadoop2</value>
  </property>
  <!-- 环境变量的继承 -->
      <property>
          <name>yarn.nodemanager.env-whitelist</name>
          <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
  </property>
  <!-- yarn容器允许分配的最大最小内存 -->
      <property>
          <name>yarn.scheduler.minimum-allocation-mb</name>
          <value>512</value>
      </property>
      <property>
          <name>yarn.scheduler.maximum-allocation-mb</name>
          <value>4096</value>
  </property>
  <!-- yarn容器允许管理的物理内存大小 -->
      <property>
          <name>yarn.nodemanager.resource.memory-mb</name>
          <value>4096</value>
  </property>
  <!-- 关闭yarn对物理内存和虚拟内存的限制检查 -->
      <property>
          <name>yarn.nodemanager.pmem-check-enabled</name>
          <value>false</value>
      </property>
      <property>
          <name>yarn.nodemanager.vmem-check-enabled</name>
          <value>false</value>
      </property>
      
      <!-- 开启日志聚集功能 -->
      <property>
          <name>yarn.log-aggregation-enable</name>
          <value>true</value>
      </property>
      <!-- 设置日志聚集服务器地址 -->
      <property>  
          <name>yarn.log.server.url</name>  
          <value>http://hadoop1:19888/jobhistory/logs</value>
      </property>
      <!-- 设置日志保留时间为7天 -->
      <property>
          <name>yarn.log-aggregation.retain-seconds</name>
          <value>604800</value>
      </property>
  <!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
  <property>
       <name>yarn.nodemanager.pmem-check-enabled</name>
       <value>false</value>
  </property>
  
  <!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
  <property>
       <name>yarn.nodemanager.vmem-check-enabled</name>
       <value>false</value>
  </property>
  
  </configuration>
  ```

  mapred-site.xml

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  
  <configuration>
  	<!-- 指定MapReduce程序运行在Yarn上 -->
      <property>
          <name>mapreduce.framework.name</name>
          <value>yarn</value>
      </property>
      <!-- 历史服务器端地址 -->
      <property>
          <name>mapreduce.jobhistory.address</name>
          <value>hadoop1:10020</value>
      </property>
  
      <!-- 历史服务器web端地址 -->
      <property>
          <name>mapreduce.jobhistory.webapp.address</name>
          <value>hadoop1:19888</value>
      </property>
  </configuration>
  ```

  配置workers

  ```
  vim $HADOOP_HOME/etc/hadoop/workers
  ```

  ```
  hadoop1
  hadoop2
  hadoop3
  ```

- 启动集群

  ```
  # 格式化节点 报错删除data和logs目录
  hdfs namenode -format
  
  # 启动HDFS  网页NN端口9870
  sbin/start-dfs.sh
   
  # 启动yarn  网页端口8088
  sbin/start-yarn.sh
  ```

  集群测试

  ```
  hadoop fs -mkdir /input
  hadoop fs -put $HADOOP_HOME/wcinput/word.txt /input
  ```

  启动脚本
  
  ```
  #!/bin/bash
  if [ $# -lt 1 ]
  then
      echo "No Args Input..."
      exit ;
  fi
  case $1 in
  "start")
          echo " =================== 启动 hadoop集群 ==================="
  
          echo " --------------- 启动 hdfs ---------------"
          ssh hadoop1 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
          echo " --------------- 启动 yarn ---------------"
          ssh hadoop2 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
          echo " --------------- 启动 historyserver ---------------"
          ssh hadoop3 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
  ;;
  "stop")
          echo " =================== 关闭 hadoop集群 ==================="
  
          echo " --------------- 关闭 historyserver ---------------"
          ssh hadoop1 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
          echo " --------------- 关闭 yarn ---------------"
          ssh hadoop2 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
          echo " --------------- 关闭 hdfs ---------------"
          ssh hadoop3 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
  ;;
  *)
      echo "Input Args Error..."
  ;;
  esac
  ```
  
  
