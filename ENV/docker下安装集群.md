## 安装

1. wsl安装
2. Win10应用商店安装terminal
3. 安装visual code，用命令```code .```就能启动visual code，针对wsl需要安装插件

root用户linux无法直接用code . 启动，需退出

4. 安装docker 注意用sudo

   开启自启

   ```sudo systemctl enable docker```

5. 安装zsh

   ```sudo apt install zsh```

   配置相关

   https://github.com/ohmyzsh/ohmyzsh

   下载脚本后安装

   <!--more-->

   ```
   sh install.sh
   ```

6. 配置ssh

   因为windows10默认占用了22的端口，因此需将ssh端口改为其他端口

   ```linux
   # 将端口改为2222
   sudo sed -i '/Port /c Port 2222' /etc/ssh/sshd_config
   
   # 修改ssh的监听地址
   sudo sed -i '/ListenAddress 0.0.0.0/c ListenAddress 0.0.0.0' /etc/ssh/sshd_config
   
   # 重启ssh服务
   sudo service ssh restart
   
   # 重启服务出现以下错误
   sshd: no hostkeys available -- exiting.
   
   # 解决方法 配置hostkeys
   sudo ssh-keygen -A
   
   # 配置信息
   sudo vim /etc/ssh/sshd_config
   
   # 为了避免与windows的ssh服务冲突，这里端口务必修改
   Port 2222 
   # Privilege Separation is turned on for security
   UsePrivilegeSeparation no
   # 登陆验证
   PasswordAuthentication yes
   
   # 重启服务
   sudo service ssh restart
   ```

   

## docker 搭建centos Hadoop单点伪分布式

编辑Dockerfile文件

```
vim Dockerfile

# 添加如下信息

#选择centos7.9作为基础镜像
FROM centos:centos7.9.2009
#描述信息
LABEL name="Hadoop-Single" \
            build_date="2021-10-31 11:24:12"
#构建容器时需要运行的命令
#安装openssh-server、openssh-clients、sudo、vim和net-tools软件包
RUN yum -y install openssh-server openssh-clients sudo vim net-tools
#生成相应的主机密钥文件
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key
RUN ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
#创建自定义组和用户、设置密码并授予root权限
RUN groupadd -g 1124 bigdata && useradd -m -u 1124 -g bigdata llc
RUN echo "llc:passwd" | chpasswd
RUN echo "llc    ALL=(ALL)   NOPASSWD:ALL" >> /etc/sudoers 
#创建模块和软件目录并修改权限
RUN mkdir /opt/software && mkdir /opt/moudle
#将宿主机的文件拷贝至镜像（ADD会自动解压）
COPY copyright.txt /home/llc/copyright.txt
ADD jdk-8u212-linux-x64.tar.gz /opt/moudle
ADD hadoop-3.1.3.tar.gz /opt/software
RUN chown -R llc:bigdata /opt/moudle && chown -R llc:bigdata /opt/software
#设置环境变量
ENV CENTOS_DEFAULT_HOME /root
ENV JAVA_HOME /opt/moudle/jdk1.8.0_212
ENV HADOOP_HOME /opt/software/hadoop-3.1.3
ENV JRE_HOME ${JAVA_HOME}/jre
ENV CLASSPATH ${JAVA_HOME}/lib:${JRE_HOME}/lib
ENV PATH ${JAVA_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:$PATH
#终端默认登录进来的工作目录
WORKDIR $CENTOS_DEFAULT_HOME
#启动sshd服务并且暴露22端口 
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```



```linux
# 创建镜像
sudo docker build -f Hadoop-Single-Dockerfile -t llc/hadoop-pseudo:3.1.3 .
# 查看镜像
docker images
```

网络设置，默认桥接模式，每次启动更改ip，需自定义网络

```linux
# 查看支持的网络
docker network ls
-------------
NETWORK ID     NAME      DRIVER    SCOPE
0f5d41d331b5   bridge    bridge    local
abee4898156b   host      host      local
0880589c3ba0   none      null      local
-------------

# 查看网络信息
 docker network inspect 0f5d41d331b5
--------------
"Config": [
{
"Subnet": "172.17.0.0/16",
"Gateway": "172.17.0.1"
}
---------------
# 创建子网，与bridge区别
sudo docker network create --subnet=172.22.0.0/24 mynetwork
```

启动容器

```
docker images
----------
REPOSITORY          TAG              IMAGE ID       CREATED          SIZE
llc/hadoop-pseudo   3.1.3            c404d7d9b8ab   17 minutes ago   2.91GB
centos              centos7.9.2009   eeb6ee3f44bd   6 weeks ago      204MB
----------

sudo docker run -d --name hadoop --hostname hadoop -P -p 9870:9870 -p 8088:8088 -p 19888:19888 --net mynetwork --ip 172.22.0.2 c404d7d9b8ab
```

-d 守护进程

-P端口映射

--net网络



进入容器

```linux
sudo docker exec -it --privileged=true 2accebd56642 /bin/bash
```

-ti 终端交互输入方式

在容器中配置

配置免密登录

```
su llc
ssh-keygen -t rsa -C "username@domain.com"
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop

# 测试
ssh hadoop
```

创建所需文件

```
mkdir /opt/software/hadoop-3.1.3/tmp
mkdir -p /opt/software/hadoop-3.1.3/dfs/namenode_data
mkdir -p /opt/software/hadoop-3.1.3/dfs/datanode_data
```

查看java和hadoop配置是否正确

```
cd $HADOOP_HOME
java -version
```

切换目录修改配置信息

```
cd ${HADOOP_HOME}/etc/hadoop
```

修改hdaoop-env.sh

```
export JAVA_HOME=/opt/moudle/jdk1.8.0_212
export HADOOP_CONF_DIR=/opt/software/hadoop-3.1.3
```

配置core-site.xml

```xml
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop:8020</value>
        </property>
        <!--指定HDFS执行时的临时目录 -->
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/software/hadoop-3.1.3/tmp</value>
        </property>
```

配置hdfs-site

```xml
        <property>
                <!--指定hdfs保存数据副本的数量，包括自己，默认为3-->
                <!--伪分布式模式，此值必须为1-->
                <name>dfs.replication</name>
                <value>1</value>
        </property>
        <property>
                <!--namenode节点数据（元数据）的存放位置,可以指定多个目录实现容错，用逗号分隔
-->
                <name>dfs.name.dir</name>
                <value>/opt/software/hadoop-3.1.3/dfs/namenode_data</value>
        </property>
        <property>
                <!--datanode节点数据（数据块）的存放位置-->
                <name>dfs.datanode.data.dir</name>
                <value>/opt/software/hadoop-3.1.3/dfs/datanode_data</value>
        </property>
        <property>
                <!--设置hdfs操作权限，false表示任何用户都可以在hdfs上操作文件-->
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
```

配置mapred-site

```xml
		<property>
            <!--指定mapreduce运行在yarn上-->
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <!--配置任务历史服务器地址-->
            <name>mapreduce.jobhistory.address</name>
            <value>hadoop:10020</value>
        </property>
        <property>
            <!--配置任务历史服务器web-UI地址-->
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>hadoop:19888</value>
        </property>
```

配置yarn-site

```xml
        <property>
                <!--指定yarn的老大resourcemanager的地址-->
                <name>yarn.resourcemanager.hostname</name>
                <value>hadoop</value>
        </property>
        <property>
                <!--NodeManager获取数据的方式-->
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
            <!--开启日志聚集功能-->
            <name>yarn.log-aggregation-enable</name>
            <value>true</value>
        </property>
        <property>
            <!--配置日志保留7天-->
            <name>yarn.log-aggregation.retain-seconds</name>
            <value>604800</value>
        </property>
```

配置works

```linux
vim ${HADOOP_HOME}/etc/hadoop/workers

# 加入下面内容
hadoop
```

退出容器

```
exit
```



关闭防火墙

```linux
# 查看防火墙状态
systemctl status firewalld
# 关闭防火墙:
sudo systemctl stop firewalld.service
```

启动hdfs

```linux
# 格式化
hdfs namenode -format

# 启动
start-dfs.sh
start-yarn.sh

# 查看是否启动成功
jps
----------
2674 Jps
1187 DataNode
2311 JobHistoryServer
1415 SecondaryNameNode
1721 ResourceManager
1020 NameNode
2045 NodeManager
----------

# 可以在windows下访问了，注意是宿主ip地址
ifconfig
-----------
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.26.81.41  netmask 255.255.240.0  broadcast 172.26.95.255
-----------

```

## docker镜像提交备份与恢复

将运行中的容器提交为镜像 ---> 镜像备份 ---> 镜像恢复

镜像提交

```
sudo docker commit -a "llc" -m "hadoop backup" 2accebd56642 llc/hadoop-commit:3.1.3
```

-a 作者

-m 注释

创建备份

```
sudo docker save -o hadoop-commit-pseudo.tar.gz llc/hadoop-commit:3.1.3
```

恢复镜像

```
sudo docker load -i hadoop-commit-pseudo.tar.gz
```



## 常见问题解决

```linux
# 创建组
sudo groupadd docker

# 添加用户到创建的组中
sudo gpasswd -a {your user name of linux} docker

# 重启docker-daemon
# centos用
sudo systemctl restart docker
# ubuntu用
service docker restart

# 更新组
newgrp docker

# 设置目录及其所有文件的权限
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R
```

重启Linux方法，重启前需关闭桌面的docker，管理员启动cmd

```
net stop LxssManager	//停止
net start LxssManager	//启动
```

## docker搭建完全分布式

启动脚本

```shell
vim hadoop-3.1.3.sh

#!/bin/bash
# description:Batch start Containers Script
# author:xiaokang
if [ $# -ne 1  ];then
        echo "You need to start several containers explicitly."
        echo "Some like './hadoop-cluster.sh 3' or 'sh hadoop-cluster.sh 3'"
        exit 1
fi
#要启动的容器数量
NUM_CONTAINERS=$1
#自定义网络名称
NETWORK_NAME=mynetwork
#镜像ID
IMAGE_ID=c404d7d9b8ab
#前缀
PREFIX="0"
for i in `seq $NUM_CONTAINERS`
# for (( i=1;i<=$NUM_CONTAINERS;i++ )) 出错的地方
        do
        	if [ $i -eq 1  ];then
        	# $[$i+1] 出错 改为$(( i+1 ))
                	sudo docker run -d --name hadoop$PREFIX$i --hostname hadoop$PREFIX$i --net ${NETWORK_NAME} --ip 172.22.0.$(( i+1 )) -P -p 9870:9870 -p 8088:8088 -p 19888:19888 --privileged $IMAGE_ID /usr/sbin/init
             elif [ $i -eq 2 ];then
                 sudo docker run -d --name hadoop$PREFIX$i --hostname hadoop$PREFIX$i --net ${NETWORK_NAME} --ip 172.22.0.$(( i+1 )) -P -p 9870:9870 --privileged $IMAGE_ID /usr/sbin/init
             else
                 sudo docker run -d --name hadoop$PREFIX$i --hostname hadoop$PREFIX$i --net ${NETWORK_NAME} --ip 172.22.0.$(( i+1 )) -P --privileged $IMAGE_ID /usr/sbin/init
             fi
        done
echo "$NUM_CONTAINERS containers started!"
echo "==================================="
sudo docker ps | grep hadoop
echo "============IPv4==================="
sudo docker inspect $(sudo docker ps -q) | grep -i ipv4
```

启动脚本

```
sh hadoop-3.1.3.sh 4

# 显示所有容器ID
docker container ls -aq

# 删除所有容器
docker rm $(docker container ls -aq)
```

启动对应容器

```
su llc
sudo docker exec -ti 3d03ab4fc90f  /bin/bash


 sudo docker run -d --name hadoop02 --hostname hadoop02 --net mynetwork --ip 172.22.0.03 -P --privileged c404d7d9b8ab /usr/sbin/init
```

配置hosts

```
hadoop01=172.22.0.2
hadoop02=172.22.0.3
hadoop03=172.22.0.4
hadoop04=172.22.0.5
```



```
# 生成密钥
ssh-keygen -t rsa -N '' -C "usename@domain.com"

# 复制公钥
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop01
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop02
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop03
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop04
```

主节点配置

```
mkdir /opt/software/hadoop-3.1.3/tmp
mkdir -p /opt/software/hadoop-3.1.3/dfs/namenode_data
mkdir -p /opt/software/hadoop-3.1.3/dfs/datanode_data
mkdir -p /opt/software/hadoop-3.1.3/checkpoint/dfs/cname
```

hadoop-env.sh

```xml
# 添加如下信息
export HADOOP_CONF_DIR=/opt/software/hadoop-3.1.3/etc/hadoop
export JAVA_HOME=/opt/moudle/jdk1.8.0_212
```

core-site.xml

```xml
        <property>
            <!--用来指定hdfs的老大，namenode的地址-->
            <name>fs.defaultFS</name>
            <value>hdfs://hadoop01:8020</value>
        </property>  
        <property>
            <!--用来指定hadoop运行时产生文件的存放目录-->   
            <name>hadoop.tmp.dir</name>
            <value>/opt/software/hadoop-3.1.3/tmp</value>
        </property>
        <property>
            <!--设置缓存大小，默认4kb-->
            <name>io.file.buffer.size</name>
            <value>4096</value>
        </property>
```

hdfs-site.xml

```xml
        <property>
           <!--数据块默认大小128M-->
           <name>dfs.block.size</name>
           <value>134217728</value>
        </property>
        <property>
            <!--副本数量，不配置的话默认为3-->
            <name>dfs.replication</name> 
            <value>3</value>
        </property>
        <property>
                 <!--定点检查--> 
                 <name>fs.checkpoint.dir</name>
                 <value>/opt/software/hadoop-3.1.3/checkpoint/dfs/cname</value>
        </property>
        <property>
            <!--namenode节点数据（元数据）的存放位置-->
            <name>dfs.name.dir</name> 
            <value>/opt/software/hadoop-3.1.3/dfs/namenode_data</value>
        </property>
        <property>
            <!--datanode节点数据（元数据）的存放位置-->
            <name>dfs.data.dir</name> 
            <value>/opt/software/hadoop-3.1.3/dfs/datanode_data</value>
        </property>
        <property>
            <!--指定secondarynamenode的web地址-->
            <name>dfs.namenode.secondary.http-address</name> 
            <value>hadoop02:50090</value>
        </property>
        <property>
            <!--hdfs文件操作权限,false为不验证-->
            <name>dfs.permissions</name> 
            <value>false</value>
        </property>
```

mapred-site.xml

```xml
        <property>
            <!--指定mapreduce运行在yarn上-->
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <!--配置任务历史服务器地址-->
            <name>mapreduce.jobhistory.address</name>
            <value>hadoop01:10020</value>
        </property>
        <property>
            <!--配置任务历史服务器web-UI地址-->
            <name>mapreduce.jobhistory.webapp.address</name>
            <value>hadoop01:19888</value>
        </property>
```



```linux
        <property>
        <!--指定yarn的老大resourcemanager的地址-->
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop01</value>
        </property>
        <property>
            <name>yarn.resourcemanager.address</name>
            <value>hadoop01:8032</value>
        </property>
        <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>hadoop01:8088</value>
        </property>
        <property>
            <name>yarn.resourcemanager.scheduler.address</name>
            <value>hadoop01:8030</value>
        </property>
        <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>hadoop01:8031</value>
        </property>
        <property>
            <name>yarn.resourcemanager.admin.address</name>
            <value>hadoop01:8033</value>
        </property>
        <property>
            <!--NodeManager获取数据的方式-->
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <!--开启日志聚集功能-->
            <name>yarn.log-aggregation-enable</name>
            <value>true</value>
        </property>
        <property>
            <!--配置日志保留7天-->
            <name>yarn.log-aggregation.retain-seconds</name>
            <value>604800</value>
        </property>
```



分发数据

```
sudo scp -r /opt/software/hadoop-3.1.3/ llc@hadoop02:/opt/software/
```



```
sudo docker commit -a "llc" -m "hadoop01 backup" 3d03ab4fc90f llc/hadoop01-commit:3.1.3
sudo docker commit -a "llc" -m "hadoop02 backup" 382312b2f963 llc/hadoop02-commit:3.1.3
sudo docker commit -a "llc" -m "hadoop03 backup" 2d937f399415 llc/hadoop03-commit:3.1.3
sudo docker commit -a "llc" -m "hadoop04 backup" 5b19409d392c llc/hadoop04-commit:3.1.3

sudo docker save -o hadoop01-commit-pseudo.tar.gz llc/hadoop01-commit:3.1.3
sudo docker save -o hadoop02-commit-pseudo.tar.gz llc/hadoop02-commit:3.1.3
sudo docker save -o hadoop03-commit-pseudo.tar.gz llc/hadoop03-commit:3.1.3
sudo docker save -o hadoop04-commit-pseudo.tar.gz llc/hadoop04-commit:3.1.3
```



测试

```linux
hadoop jar /opt/software/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar pi 11 24
```

