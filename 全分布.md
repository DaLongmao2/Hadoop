## 一、防火墙(都执行)

~~~shell
# 1.进入 root
$[user] su root
# 2.永久关闭防火墙
$[root] systemctl disable firewalld
~~~

## 二、JDK(都执行)

~~~shell
# 1.卸载自带JDK
$[root] rpm -qa | grep java
$[root] rpm -e --nodeps ...

# 2.安装 JDK
$[root]　tar -zxvf /home/zhang/jdk-8u152-linux-x64.tar.gz -C /usr/local/src

# 3.重命名 JDK
$[root] mv /usr/local/src/jdk-8u152 /usr/local/src/jdk

# 4.配置 JDK
$[root] vim /etc/profile
	- export JAVA_HOME=/usr/local/src/jdk
	- export PATH=$PATH:$JAVA_HOME/bin
	
# 5.生效
$[root] source /etc/profile

# 6.校验
$[root] java -version
~~~

## 三、设置主机 配置集群(都执行)

~~~  shell
# 1.设置主机名称
$[root] hostnamectl set-hostname [master/slave1/slave2]

# 2.配置主机映射
$[root] vim /etc/hosts
	- 192.168.1.6 master
	- 192.168.1.7 slave1
	- 192.168.1.8 slave2
	
# 3.生产密钥  ***普通用户下 普通用户下 普通用户下 普通用户下 普通用户下 普通用户下 普通用户下
$[root] su - zhang

# 4.三台逐一生成 在 逐一 互换密钥
$[zhang] ssh-keygen -t rsa

# 5.逐一生成结束后 互相copy密钥 （先copy给自己 在逐一copy给别人）

$[zhang] ssh-copy-id -i ~/.ssh/id_rsa.pub [master/slave1/slave2]

# 6.修改权限
$[zhang] chmod 600 ~/.ssh/authorized_keys

# 7.配置 SSH 服务  *** Root下 Root下 Root下 Root下 Root下 Root下 Root下 Root下
$ [root] vim /etc/ssh/sshd_config
	# 取消该行的注释
	- PubkeyAuthentication yes 
	
# 8.验证   ***普通用户下 普通用户下 普通用户下 普通用户下 普通用户下 普通用户下 普通用户下
$[zhang] ssh slave1
$[zhang] ssh slave2
~~~

## 四、安装 配置Hadoop  (Master下执行！！！)

~~~ shell
# 1.解压Hadoop
$[root] tar -zxvf /opt/software/hadoop-2.7.1.tar.gz -C /usr/local/src/

# 2.重命名
$[root] mv /usr/local/src/hadoop-2.7.1 /usr/local/src/hadoop

# 3.配置 Hadoop 环境变量
$[root] vim /etc/profile
	- export HADOOP_HOME=/usr/local/src/hadoop
	- export PATH=$HADOOP_HOME/bin
	- export PATH=$PATH:$HADOOP_HOME/sbin

# 4.生效
source /etc/profile

# 进入hadoop etc 路径 配置
$[root] cd /usr/local/src/hadoop/etc/hadoop

# 6. hadoop-env.sh 配置
$[root] vim hadoop-env.sh
	- export JAVA_HOME=/usr/local/src/jdk1.8.0_152
	
# 7. hdfs-site.xml 配置
$[root] vim hdfs-site.xml
<property>
<name>dfs.namenode.name.dir</name>
<value>file:/usr/local/src/hadoop/dfs/name</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>file:/usr/local/src/hadoop/dfs/data</value>
</property>
<property>
<name>dfs.replication</name>
<value>3</value>
</property>

# 8. core-site.xml 配置
<property>
<name>fs.defaultFS</name>
<value>hdfs://master:9000</value>
</property>
<property>
<name>io.file.buffer.size</name>
<value>131072</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>file:/usr/local/src/hadoop/tmp</value>
</property>

# 9.mapred-site.xml 配置
$[root] cp mapred-site.xml.template mapred-site.xml
$[root] vim mapred-site.xml
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
<property>
<name>mapreduce.jobhistory.address</name>
<value>master:10020</value>
</property>
<property>
<name>mapreduce.jobhistory.webapp.address</name>
<value>master:19888</value>
</property>

# 10.yarn-site.xml 配置
<property>
<name>yarn.resourcemanager.address</name>
<value>master:8032</value>
</property>
<property>
<name>yarn.resourcemanager.scheduler.address</name>
<value>master:8030</value>
</property>
<property>
<name>yarn.resourcemanager.resource-tracker.address</name>
<value>master:8031</value>
</property>
<property>
<name>yarn.resourcemanager.admin.address</name>
<value>master:8033</value>
</property>
<property>
<name>yarn.resourcemanager.webapp.address</name>
<value>master:8088</value>
</property>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.aux-
services.mapreduce.shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>

# 11.配置 masters 文件
$[root] vim masters
	- master

# 12.配置 slaves 文件
$[root] vim slaves
	- master
	- slave1
	- slave2

# 13.新建目录
$[root] mkdir /usr/local/src/hadoop/tmp
$[root] mkdir /usr/local/src/hadoop/dfs/name -p
$[root] mkdir /usr/local/src/hadoop/dfs/data -p

# 14.修改目录权限
$[root] chown -R hadoop:hadoop /usr/local/src

# 15.其他节点同步
$[root] scp -r /usr/local/src/hadoop/ root@slave1:/usr/local/src/
$[root] scp -r /usr/local/src/hadoop/ root@slave2:/usr/local/src/

# 16. 以下操作在 slave1 和 slave2 中分别执行：  *** slave不是master 
$[root] vim /etc/profile
	- export HADOOP_HOME=/usr/local/src/hadoop
	- export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
$[root] chown -R hadoop:hadoop /usr/local/src/

$[root] source /etc/profile
~~~

## hadoop 集群运行

~~~ shell
$ [master root] su – zhang
$ [master zhang] cd /usr/local/src/hadoop/
# 初始化
$ [master zhang] bin/hdfs namenode –format
	# 结果： 20/05/02 16:21:50 INFO namenode.NameNode: SHUTDOWN_MSG:
$ [master zhang] jps
	# 结果： 3557 NameNode
	#       3624 Jps
	
	
	
$ [slave root] su - zhang
$ [slave zhang] cd /usr/local/src/hadoop/
# 启动 DataNode
$ [slave zhang] hadoop-daemon.sh start datanode
$ [slave zhang] jps
	# 结果：3557 DataNode
	#	   3725 Jps
	
	
# 启动 SecondaryNameNode
$ [master zhang] hadoop-daemon.sh start secondarynamenode
$ [master zhang] jps
	# 结果：34257 NameNode
	#	   34449 SecondaryNameNode
	#	   34494 Jps

# 命令查看报告
$ [master zhang] hdfs dfsadmin -report

# 网页查看
http://master:50070
http://master:50090
http://master:8088
~~~

# # Hive(Master)

~~~ shell
# 1.解压 HIVE
$[root] tar -zxvf /opt/software/apache-hive-2.0.0-bin.tar.gz -C /usr/local/src

# 2.重命名
$[root] mv /usr/local/src/apache-hive-2.0.0-bin /usr/local/src/hive

# 3.设置 Hive 环境
$[root] vim /etc/profile
	- export HIVE_HOME=/usr/local/src/hive
	- export PATH=$PATH:$HIVE_HOME/bin

# 4.生效
$[root] source /etc/profile

# 5.卸载 MariaDB 数据库
$[root] rpm -qa | grep mariadb
		rpm -e --nodeps mariadb-....
		
# 6.安装mysql数据库
$[root] rpm -ivh mysql-community-common-5.7.18-1.el7.x86_64.rpm
		rpm -ivh mysql-community-libs-5.7.18-1.el7.x86_64.rpm
		rpm -ivh mysql-community-client-5.7.18-1.el7.x86_64.rpm
		rpm -ivh mysql-community-server-5.7.18-1.el7.x86_64.rpm
		
# 7.配置 /etc/my.cnf
$[root] vim /etc/my.cnf
    default-storage-engine=innodb
    innodb_file_per_table
    collation-server=utf8_general_ci
    init-connect='SET NAMES utf8'
    character-set-server=utf8
    
# 8.启动 MySQL 数据库
$[root] systemctl start mysqld

# 9.查看密码
$[root] cat /var/log/mysqld.log | grep password

# 10.MySQL 数据库初始化
$[root] mysql_secure_installation
	1.设置密码
	2.输入5次 Y
	 
# 11. 进入mysql
$[root] mysql -uroot -p[password]
@ZhangYi520
# 添加 root 用户本地访问授权
mysql>> grant all privileges on *.* to root@'localhost'identified by '[新密码]';
# 添加 root 用户远程访问授权
mysql>> grant all privileges on *.* to root@'%' identified by '[新密码]';
# 刷新授权
mysql>> flush privileges;
# 查看
mysql>> select user,host from mysql.user where user='root';
# 退出
mysql>> exit

# 12.修改 Hive 组件配置文件。 *** 普通用户！！！！！！！！！
$[root] su - zhang


$[zhang] vim /usr/local/src/hive/conf/hive-site.xml
	490 --> jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false
	475 --> [Mysql 密码]
	656 --> flase
	881 --> com.mysql.jdbc.Driver
	906 --> root
	1436 --> /usr/local/src/hive/tmp
	43 --> /usr/local/src/hive/tmp
	49 --> /usr/local/src/hive/tmp/resources
	3424 --> /usr/local/src/hive/tmp/operation_logs

# 13.创建临时文件夹
$[zhang] mkdir /usr/local/src/hive/tmp

# 14.复制mysql驱动
$[zhang] cp /opt/software/mysql-connector-java-5.1.46.jar /usr/local/src/hive/lib/

# 15.重启Hadoop
$[zhang] stop-all.sh
		 start-all.sh
		 
# 16.初始化数据库
$[zhang] schematool -initSchema -dbType mysql

# 17.启动Hive！！！！！！！！！！
$[zhang] hive
~~~

