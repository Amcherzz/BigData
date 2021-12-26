## Hive安装记录

- 检查系统是否安装MySQL

  ```linux
  rpm -qa|grep mariadb
  
  # 如果存在就删除
  sudo rpm -e --nodeps  mariadb-libs   //用此命令卸载mariadb
  ```

  

- 安装MySQL

  ```
  mkdir mysql-lib
  tar -xf mysql-5.7.28-1.el7.x86_64.rpm-bundle.tar -C mysql-lib
  cd mkdir mysql-lib
  sudo rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
  sudo rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
  sudo rpm -ivh mysql-community-libs-compat-5.7.28-1.el7.x86_64.rpm
  sudo rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
  
  # 如果安装以下命令出现错误
  sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
  ```

  ```
  warning: mysql-community-server-5.7.28-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
  error: Failed dependencies:
          libnuma.so.1()(64bit) is needed by mysql-community-server-5.7.28-1.el7.x86_64
          libnuma.so.1(libnuma_1.1)(64bit) is needed by mysql-community-server-5.7.28-1.el7.x86_64
          libnuma.so.1(libnuma_1.2)(64bit) is needed by mysql-community-server-5.7.28-1.el7.x86_64
  ```

  解决方法可以

  ```
  # 安装libaio
  yum install -y libaio
  yum -y install numactl
  # 再执行安装
  sudo rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm --force --nodeps
  ```

  删除数据库中的内容，以便初始化

  ```
  cat /etc/my.cnf
  # datadir 为数据库路径
  [mysqld]
  datadir=/var/lib/mysql
  
  # 删除上述文件夹数据 注意
  rm -rf /var/lib/mysql/*
  
  # 初始化数据库
  sudo mysqld --initialize --user=mysql
  
  # 查看临时密码，需暂时记录一下，后续登录使用
  sudo cat /var/log/mysqld.log
  
  
  # 启动mysqld服务
  sudo systemctl start mysqld
  
  # 查看状态
  sudo systemctl status mysqld
  
  # 登录MySQL
  mysql -uroot -p
  
  Enter password:   输入临时生成的密码
  
  # 设置密码
  mysql> set password = password("123456");
  
  # 允许root任意方式连接
  mysql> update mysql.user set host='%' where user='root';
  mysql> flush privileges;
  ```

- 安装Hive

  ```
  # 安装相关文件 重命名文件夹为hive 例如：
  tar -zxvf /opt/software/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
  mv /opt/module/apache-hive-3.1.2-bin/ /opt/module/hive
  
  # 添加环境变量
  sudo vim /etc/profile.d/my_env.sh
  
  # 加入如下内容
  
  #HIVE_HOME
  export HIVE_HOME=/opt/module/hive
  export PATH=$PATH:$HIVE_HOME/bin
  
  # 更新环境变量
  source /etc/profile
  
  # 解决jar包日志冲突
  mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.bak
  ```

- Hive配置元数据到Mysql

  ```
  # 拷贝jdbc驱动
  cp /opt/software/mysql-connector-java-5.1.37.jar $HIVE_HOME/lib
  
  # 新建配置文件
  vim $HIVE_HOME/conf/hive-site.xml
  
  # 加入如下信息
  ```

  ```xml
  <?xml version="1.0"?>
  <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
  <configuration>
      <!-- jdbc连接的URL -->
      <property>
          <name>javax.jdo.option.ConnectionURL</name>
          <value>jdbc:mysql://hadoop1:3306/metastore?useSSL=false</value>
  </property>
  
      <!-- jdbc连接的Driver-->
      <property>
          <name>javax.jdo.option.ConnectionDriverName</name> 
          <value>com.mysql.jdbc.Driver</value>
  </property>
  
  	<!-- jdbc连接的username-->
      <property>
          <name>javax.jdo.option.ConnectionUserName</name>
          <value>root</value>
      </property>
  
      <!-- jdbc连接的password -->
      <property>
          <name>javax.jdo.option.ConnectionPassword</name>
          <value>123456</value>
      </property>
      <!-- Hive默认在HDFS的工作目录 -->
      <property>
          <name>hive.metastore.warehouse.dir</name>
          <value>/user/hive/warehouse</value>
      </property>
     
      <!-- 指定hiveserver2连接的端口号 -->
      <property>
          <name>hive.server2.thrift.port</name>
          <value>10000</value>
      </property>
     <!-- 指定hiveserver2连接的host -->
      <property>
          <name>hive.server2.thrift.bind.host</name>
          <value>hadoop01</value>
  </property>
  
      <!-- 指定存储元数据要连接的地址 -->
      <property>
          <name>hive.metastore.uris</name>
          <value>thrift://hadoop1:9083</value>
      </property>
      <!-- 元数据存储授权  jdbc方式想要连接元数据得关闭-->
      <property>
          <name>hive.metastore.event.db.notification.api.auth</name>
          <value>false</value>
  </property>
  <!-- Hive元数据存储版本的验证 jdbc方式想要连接元数据得关闭-->
      <property>
          <name>hive.metastore.schema.verification</name>
          <value>false</value>
  </property>
  <!-- hiveserver2的高可用参数，开启此参数可以提高hiveserver2的启动速度 -->
  <property>
      <name>hive.server2.active.passive.ha.enable</name>
      <value>true</value>
  </property>
  </configuration>
  ```
  
- 启动Hive

  ```
  # 初始化数据库
  mysql -uroot -p123456
  
  # 新建元数据库
  mysql> create database metastore;
  mysql> quit;
  
  # 初始化元数据库
  schematool -initSchema -dbType mysql -verbose
  ```

  

- 启动metastore和hiveserver2

  ```
  nohup hive --service metastore>log.txt 2>&1 &
  nohup hive --service hiveserver2>log2.txt 2>&1 &
  ```

  或者将如下脚本放在$HIVE_HOME/bin下

  ```shell
  #!/bin/bash
  HIVE_LOG_DIR=$HIVE_HOME/logs
  if [ ! -d $HIVE_LOG_DIR ]
  then
  	mkdir -p $HIVE_LOG_DIR
  fi
  #检查进程是否运行正常，参数1为进程名，参数2为进程端口
  function check_process()
  {
      pid=$(ps -ef 2>/dev/null | grep -v grep | grep -i $1 | awk '{print $2}')
      ppid=$(netstat -nltp 2>/dev/null | grep $2 | awk '{print $7}' | cut -d '/' -f 1)
      echo $pid
      [[ "$pid" =~ "$ppid" ]] && [ "$ppid" ] && return 0 || return 1
  }
  
  function hive_start()
  {
      metapid=$(check_process HiveMetastore 9083)
      cmd="nohup hive --service metastore >$HIVE_LOG_DIR/metastore.log 2>&1 &"
      cmd=$cmd" sleep 4; hdfs dfsadmin -safemode wait >/dev/null 2>&1"
      [ -z "$metapid" ] && eval $cmd || echo "Metastroe服务已启动"
      server2pid=$(check_process HiveServer2 10000)
      cmd="nohup hive --service hiveserver2 >$HIVE_LOG_DIR/hiveServer2.log 2>&1 &"
      [ -z "$server2pid" ] && eval $cmd || echo "HiveServer2服务已启动"
  }
  
  function hive_stop()
  {
      metapid=$(check_process HiveMetastore 9083)
      [ "$metapid" ] && kill $metapid || echo "Metastore服务未启动"
      server2pid=$(check_process HiveServer2 10000)
      [ "$server2pid" ] && kill $server2pid || echo "HiveServer2服务未启动"
  }
  
  case $1 in
  "start")
      hive_start
      ;;
  "stop")
      hive_stop
      ;;
  "restart")
      hive_stop
      sleep 2
      hive_start
      ;;
  "status")
      check_process HiveMetastore 9083 >/dev/null && echo "Metastore服务运行正常" || echo "Metastore服务运行异常"
      check_process HiveServer2 10000 >/dev/null && echo "HiveServer2服务运行正常" || echo "HiveServer2服务运行异常"
      ;;
  *)
      echo Invalid Args!
      echo 'Usage: '$(basename $0)' start|stop|restart|status'
      ;;
  esac
  ```
  
- 其他配置

  Hive中配置打印表头，在$HIVE_HOME/conf/hive-site.xml添加如下

  ```xml
  <property>
      <name>hive.cli.print.header</name>
      <value>true</value>
      <description>Whether to print the names of the columns in query output.</description>
  </property>
  <property>
      <name>hive.cli.print.current.db</name>
      <value>true</value>
      <description>Whether to include the current database in the Hive prompt.</description>
  </property>
  ```

- 外部访问问题

  在$HADOOP_HOME/etc/hadoop/core-site.xml 中
  
  ```xml
  <property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>*</value>
   </property>
   <property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>*</value>
  </property>
  ```
  
- 服务启动与关闭

  ```
  sudo systemctl start mysqld
  start-dfs.sh
  start-yarn.sh
  hivesvs.sh start
  
  # 关闭
  hivesvs.sh stop
  stop-all.sh
  sudo systemctl stop mysqld
  ```

- 切换为Hive on spark

  ```
  # 安装spark，配置相关文件
  tar -zxvf spark-3.0.0-bin-hadoop3.2.tgz -C /opt/module/
  
  sudo vim /etc/profile.d/my_env.sh
  
  # 添加如下内容 
  
  # SPARK_HOME
  export SPARK_HOME=/opt/module/spark
  export PATH=$PATH:$SPARK_HOME/bin
  
  # 更新配置文件
  source /etc/profile.d/my_env.sh
  
  # 新建spark配置文件
  vim $HIVE_HOME/conf/spark-defaults.conf
  
  # 加入如下内容
  
  spark.master                       yarn
  spark.eventLog.enabled             true
  spark.eventLog.dir                 hdfs://hadoop1:8020/spark-history
  spark.executor.memory              1g
  spark.driver.memory                1g
  
  # 用于创建存放日志的路径
  hadoop fs -mkdir /spark-history
  
  # 将纯净的spark包上传的集群
  tar -zxvf /opt/software/spark-3.0.0-bin-without-hadoop.tgz
  hadoop fs -mkdir /spark-jars
  hadoop fs -put spark-3.0.0-bin-without-hadoop/jars/* /spark-jars
  
  # 修改hive-site.xml文件
  vim $HIVE_HOME/conf/hive-site.xml
  ```

  添加如下内容

  ```xml
  <!--Spark依赖位置（注意：端口号8020必须和namenode的端口号一致）-->
  <property>
      <name>spark.yarn.jars</name>
      <value>hdfs://hadoop1:8020/spark-jars/*</value>
  </property>
    
  <!--Hive执行引擎-->
  <property>
      <name>hive.execution.engine</name>
      <value>spark</value>
  </property>
  
  <!--Hive和Spark连接超时时间-->
  <property>
      <name>hive.spark.client.connect.timeout</name>
      <value>10000ms</value>
  </property>
  ```

  