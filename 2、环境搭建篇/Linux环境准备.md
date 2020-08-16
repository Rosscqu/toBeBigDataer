## 1. Linux环境准备

使用三台linux服务器，来做统一的环境准备。

### 1. 三台机器IP设置

三台机器修改ip地址：

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33 

ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.xx.100
NETMASK=255.255.255.0
GATEWAY=192.168.xx.1
DNS1=8.8.8.8
```

使用service network restart命令，重启网络服务。

其中xx与本地电脑相同，这样可以使用桥接模式实现局域网其他机器访问该虚拟机。桥接模式设置参考如下：https://blog.csdn.net/xixi_hh/article/details/103106409?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param



准备三台linux机器，IP地址分别设置成为

第一台机器IP地址：192.168.xx.100

第二台机器IP地址：192.168.xx.110

第三台机器IP地址：192.168.xx.120



机器规划：

| IP             | 作用                                | 账号/密码         |
| -------------- | ----------------------------------- | ----------------- |
| 192.168.xx.100 | Namenode/datanode/secondarynamenode | hadoop/bigdata002 |
| 192.168.xx.110 | Datanode                            | hadoop/bigdata002 |
| 192.168.xx.120 | Datanode                            | hadoop/bigdata002 |



### 2. 三台机器关闭防火墙

三台机器在root用户下执行以下命令关闭防火墙

```
systemctl disable firewalld.service
```



### 3. 三台机器关闭selinux

三台机器在root用户下执行以下命令关闭selinux

三台机器执行以下命令，关闭selinux

```
vim /etc/selinux/config 

SELINUX=disabled
```



### 4. 三台机器更改主机名

三台机器分别更改主机名

第一台主机名更改为：node01

第二台主机名更改为：node02

第三台主机名更改为：node03

第一台机器执行以下命令修改主机名

```
vim /etc/hostname
node01
```

第二台机器执行以下命令修改主机名

```
vim /etc/hostname
node02
```

第三台机器执行以下命令修改主机名

```
vim /etc/hostname
node03
```



### 5. 三台机器更改主机名与IP地址映射

三台机器执行以下命令更改主机名与IP地址映射关系

```
vim /etc/hosts

192.168.52.100  node01
192.168.52.110  node02
192.168.52.120  node03
```



### 6. 三台机器同步时间

三台机器执行以下命令定时同步阿里云服务器时间

```
 yum -y install ntpdate
 crontab -e 
 */1 * * * * /usr/sbin/ntpdate time1.aliyun.com
 
 
```

### 7. 三台机器添加普通用户

三台linux服务器统一添加普通用户hadoop，并给以sudo权限，用于以后所有的大数据软件的安装

并统一设置普通用户的密码为  bigdata002

```
 useradd hadoop
 passwd hadoop
```

三台机器为普通用户添加sudo权限

```
visudo

hadoop  ALL=(ALL)       ALL
```

### 

### 8. 三台机器目录规划

定义三台linux服务器软件压缩包存放目录，以及解压后安装目录，三台机器执行以下命令，创建两个文件夹，一个用于存放软件压缩包目录，一个用于存放解压后目录

```java
 mkdir -p /kkb/soft     # 软件压缩包存放目录
 mkdir -p /kkb/install  # 软件解压后存放目录
 chown -R hadoop:hadoop /kkb    # 将文件夹权限更改为hadoop用户
```

### 9. 三台机器安装jdk

==使用hadoop用户来重新连接三台机器，然后使用hadoop用户来安装jdk软件==

上传压缩包到第一台服务器的/kkb/soft下面，然后进行解压，配置环境变量即可，三台机器都依次安装即可

```
cd /kkb/soft/

tar -zxf jdk-8u181-linux-x64.tar.gz  -C /kkb/install/
sudo vim /etc/profile


#添加以下配置内容，配置jdk环境变量
export JAVA_HOME=/kkb/install/jdk1.8.0_141
export PATH=:$JAVA_HOME/bin:$PATH
```

source /etc/profile
 java -version
问题 ：
Linux下环境变量配置错误 导致大部分命令不可以使用的解决办法
直接解决方法：在命令行中输入：export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin 后 Enter

### 10. hadoop用户免密码登录

三台机器在hadoop用户下执行以下命令生成公钥与私钥比

```
ssh-keygen -t rsa 
三台机器在hadoop用户下，执行以下命令将公钥拷贝到node01服务器上面去
ssh-copy-id  node01
node01在hadoop用户下，执行以下命令，将authorized_keys拷贝到node02与node03服务器
cd /home/hadoop/.ssh/
scp authorized_keys  node02:$PWD
scp authorized_keys  node03:$PWD

#修改.ssh目录权限
[hadoop@node01 ~]$ chmod -R 755 .ssh/
[hadoop@node01 ~]$ cd .ssh/
[hadoop@node01 .ssh]$ chmod 644 *
[hadoop@node01 .ssh]$ chmod 600 id_rsa
[hadoop@node01 .ssh]$ chmod 600 id_rsa.pub 
#把hadoop的公钥添加到本机认证文件当中，如果没有添加启动集群时需要输入密码才能启动,这里需要重点注意下
[hadoop@node01 .ssh]$ cat id_rsa.pub >> authorized_keys 
[hadoop@node01 .ssh]$
```



## 2. 安装和配置hadoop

在一台机器上配置好后复制到其他机器上即可,这样保证三台机器的hadoop配置是一致的.

下载CDH发型版的hadoop

[下载地址](http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.6.0-cdh5.14.2.tar.gz)

### 1.上传hadoop安装包,进行解压

```shell
#解压安装包到/kkb/install目录下
[root@node01 ~]# tar -xzvf hadoop-2.6.0-cdh5.14.2.tar.gz -C /kkb/install
```

### 2.配置hadoop环境变量

1.配置环境变量

```shell
#1.在linux系统全局配置文件的末尾进行hadoop和java的环境变量配置
[root@node1 ~]# vi /etc/profile
[root@node01 ~]# vi /etc/profile
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}
"/etc/profile" 85L, 2028C

for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

unset i
unset -f pathmunge

JAVA_HOME=/kkb/install/jdk1.8.0_141
HADOOP_HOME=/kkb/install/hadoop-2.6.0-cdh5.14.2

PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

export JAVA_HOME
export HADOOP_HOME
export PATH                                                                                                                                                                                                     
:wq!
```



2.验证环境变量

```shell
#1.使环境变量生效
[root@node1 ~]# source .bash_profile 
#2.显示hadoop的版本信息
[root@node1 ~]# hadoop version
#3.显示出hadoop版本信息表示安装和环境变量成功.
Hadoop 3.1.2
Source code repository https://github.com/apache/hadoop.git -r 1019dde65bcf12e05ef48ac71e84550d589e5d9a
Compiled by sunilg on 2019-01-29T01:39Z
Compiled with protoc 2.5.0
From source with checksum 64b8bdd4ca6e77cce75a93eb09ab2a9
This command was run using /opt/bigdata/hadoop-3.1.2/share/hadoop/common/hadoop-common-3.1.2.jar
[root@node1 ~]# 
```

**hadoop用户下也需要按照root用户配置环境变量的方式操作一下**



### 3.配置hadoop-env.sh

这个文件只需要配置JAVA_HOME的值即可,在文件中找到export JAVA_HOME字眼的位置，删除最前面的#

```shell
export JAVA_HOME=/kkb/install/jdk1.8.0_141
```

```shell
[root@node1 ~]#cd /kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
You have new mail in /var/spool/mail/root
[root@node1 hadoop]# pwd
/opt/bigdata/hadoop-3.1.2/etc/hadoop
[root@node1 hadoop]# vi hadoop-env.sh 
```

### 4.配置core-site.xml

切换到/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop目录下

```shell
[root@node1 ~]# cd /kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop
```

```xml
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://node01:8020</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/tempDatas</value>
	</property>
	<!--  缓冲区大小，实际工作中根据服务器性能动态调整 -->
	<property>
		<name>io.file.buffer.size</name>
		<value>4096</value>
	</property>
<property>
     <name>fs.trash.interval</name>
     <value>10080</value>
     <description>检查点被删除后的分钟数。 如果为零，垃圾桶功能将被禁用。 
     该选项可以在服务器和客户端上配置。 如果垃圾箱被禁用服务器端，则检查客户端配置。 
     如果在服务器端启用垃圾箱，则会使用服务器上配置的值，并忽略客户端配置值。</description>
</property>

<property>
     <name>fs.trash.checkpoint.interval</name>
     <value>0</value>
     <description>垃圾检查点之间的分钟数。 应该小于或等于fs.trash.interval。 
     如果为零，则将该值设置为fs.trash.interval的值。 每次检查指针运行时，
     它都会从当前创建一个新的检查点，并删除比fs.trash.interval更早创建的检查点。</description>
</property>
</configuration>
```

### 5.配置hdfs-site.xml

配置/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop目录下的hdfs-site.xml

```xml
<configuration>
	<!-- NameNode存储元数据信息的路径，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割   --> 
	<!--   集群动态上下线 
	<property>
		<name>dfs.hosts</name>
		<value>/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop/accept_host</value>
	</property>
	
	<property>
		<name>dfs.hosts.exclude</name>
		<value>/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop/deny_host</value>
	</property>
	 -->
	 
	 <property>
			<name>dfs.namenode.secondary.http-address</name>
			<value>node01:50090</value>
	</property>

	<property>
		<name>dfs.namenode.http-address</name>
		<value>node01:50070</value>
	</property>
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>file:///kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas</value>
	</property>
	<!--  定义dataNode数据存储的节点位置，实际工作中，一般先确定磁盘的挂载目录，然后多个目录用，进行分割  -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>file:///kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/datanodeDatas</value>
	</property>
	
	<property>
		<name>dfs.namenode.edits.dir</name>
		<value>file:///kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/edits</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.dir</name>
		<value>file:///kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/snn/name</value>
	</property>
	<property>
		<name>dfs.namenode.checkpoint.edits.dir</name>
		<value>file:///kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/snn/edits</value>
	</property>
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
	<property>
		<name>dfs.blocksize</name>
		<value>134217728</value>
	</property>
</configuration>
```

### 6.配置mapred-site.xml

```shell
#默认没有mapred-site.xml文件,这里需要从模板中复制一份出来进行修改配置
[root@node01 hadoop]# cp  mapred-site.xml.template mapred-site.xml
```

配置/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop目录下的mapred-site.xml

```xml
<!--指定运行mapreduce的环境是yarn -->
<configuration>
   <property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>

	<property>
		<name>mapreduce.job.ubertask.enable</name>
		<value>true</value>
	</property>
	
	<property>
		<name>mapreduce.jobhistory.address</name>
		<value>node01:10020</value>
	</property>

	<property>
		<name>mapreduce.jobhistory.webapp.address</name>
		<value>node01:19888</value>
	</property>
</configuration>
```



### 7.配置yarn-site.xml

配置/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop目录下的yarn-site.xml

```xml
<configuration>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>node01</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>

	
	<property>
		<name>yarn.log-aggregation-enable</name>
		<value>true</value>
	</property>


	<property>
		 <name>yarn.log.server.url</name>
		 <value>http://node01:19888/jobhistory/logs</value>
	</property>

	<!--多长时间聚合删除一次日志 此处-->
	<property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>2592000</value><!--30 day-->
	</property>
	<!--时间在几秒钟内保留用户日志。只适用于如果日志聚合是禁用的-->
	<property>
        <name>yarn.nodemanager.log.retain-seconds</name>
        <value>604800</value><!--7 day-->
	</property>
	<!--指定文件压缩类型用于压缩汇总日志-->
	<property>
        <name>yarn.nodemanager.log-aggregation.compression-type</name>
        <value>gz</value>
	</property>
	<!-- nodemanager本地文件存储目录-->
	<property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>/kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/yarn/local</value>
	</property>
	<!-- resourceManager  保存最大的任务完成个数 -->
	<property>
        <name>yarn.resourcemanager.max-completed-applications</name>
        <value>1000</value>
	</property>

</configuration>
```

### 8.编辑slaves

此文件用于配置集群有多少个数据节点,我们把node2，node3作为数据节点,node1作为集群管理节点.

配置/kkb/install/hadoop-2.6.0-cdh5.14.2/etc/hadoop目录下的slaves

```shell
[root@node1 hadoop]# vi slaves 
#将localhost这一行删除掉
node01
node02
node03
~                          
```

### 9.创建文件存放目录

node01机器上面创建以下目录

```shell
[root@node01 ~]# mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/tempDatas
mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas
[root@node01 ~]# mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas
[root@node01 ~]# mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/datanodeDatas
[root@node01 ~]# mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/edits
[root@node01 ~]# mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/snn/name
[root@node01 ~]# mkdir -p /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/snn/edits
[root@node01 ~]# 
```



# 3. 修改hadoop安装目录的权限

node01,node02，node03安装目录的权限

node01节点操作

```shell
#1.修改目录所属用户和组为hadoop:hadoop
[root@node01 ~]# chown -R hadoop:hadoop /kkb
[root@node01 ~]# 

#2.修改目录所属用户和组的权限值为755
[root@node01 ~]# chmod -R 755  /kkb
[root@node01 ~]# 
```

node02节点操作

```shell
#1.修改目录所属用户和组为hadoop:hadoop
[root@node02 ~]# chown -R hadoop:hadoop /kkb
#2.修改目录所属用户和组的权限值为755
[root@node02 ~]#  chmod -R 755  /kkb
[root@node02 ~]# 
```

node03节点操作

```shell
#1.修改目录所属用户和组为hadoop:hadoop
[root@node03 ~]# chown -R hadoop:hadoop /kkb
#2.修改目录所属用户和组的权限值为755
[root@node03 ~]#  chmod -R 755  /kkb

```



# 4. 格式化hadoop

```shell
#切换
[root@node01 ~]# su - hadoop
[hadoop@node01 hadoop]$  hdfs namenode -format
[hadoop@node01 ~]$ hdfs namenode -format
19/08/23 04:32:34 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   user = hadoop
STARTUP_MSG:   host = node01.kaikeba.com/192.168.52.100
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.6.0-cdh5.14.2
STARTUP_MSG:   classpath = /kkb/install/hadoop-2.6.0-19/08/23 04:32:35 INFO common.Storage: Storage directory /kkb/install/hadoop-2.6.0-
#显示格式化成功。。。
cdh5.14.2/hadoopDatas/namenodeDatas has been successfully formatted.
19/08/23 04:32:35 INFO common.Storage: Storage directory /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/dfs/nn/edits has been successfully formatted.
19/08/23 04:32:35 INFO namenode.FSImageFormatProtobuf: Saving image file /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas/current/fsimage.ckpt_0000000000000000000 using no compression
19/08/23 04:32:35 INFO namenode.FSImageFormatProtobuf: Image file /kkb/install/hadoop-2.6.0-cdh5.14.2/hadoopDatas/namenodeDatas/current/fsimage.ckpt_0000000000000000000 of size 323 bytes saved in 0 seconds.
19/08/23 04:32:35 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
19/08/23 04:32:35 INFO util.ExitUtil: Exiting with status 0
19/08/23 04:32:35 INFO namenode.NameNode: SHUTDOWN_MSG: 
#此处省略部分日志
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at node01.kaikeba.com/192.168.52.100
************************************************************/
[hadoop@node01 ~]$ 
```



# 5. 启动集群

```shell
[hadoop@node01 ~]$ start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
19/08/23 05:18:09 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Starting namenodes on [node01]
node01: starting namenode, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/hadoop-hadoop-namenode-node01.kaikeba.com.out
node01: starting datanode, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/hadoop-hadoop-datanode-node01.kaikeba.com.out
node03: starting datanode, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/hadoop-hadoop-datanode-node03.kaikeba.com.out
node02: starting datanode, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/hadoop-hadoop-datanode-node02.kaikeba.com.out
Starting secondary namenodes [node01]
node01: starting secondarynamenode, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/hadoop-hadoop-secondarynamenode-node01.kaikeba.com.out
19/08/23 05:18:24 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
starting yarn daemons
starting resourcemanager, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/yarn-hadoop-resourcemanager-node01.kaikeba.com.out
node03: starting nodemanager, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/yarn-hadoop-nodemanager-node03.kaikeba.com.out
node02: starting nodemanager, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/yarn-hadoop-nodemanager-node02.kaikeba.com.out
node01: starting nodemanager, logging to /kkb/install/hadoop-2.6.0-cdh5.14.2/logs/yarn-hadoop-nodemanager-node01.kaikeba.com.out
[hadoop@node01 ~]$ 
```

在浏览器地址栏中输入<http://192.168.xx.100:50070查看namenode的web界面.

