## Hive安装教程





### 1、Mysql安装配置



#### 1.1 mysql安装

首先安装wget命令

```shell
yum -y install wget
```

然后下载mysql

```
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

安装mysql

```
yum -y install mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql-community-server
```

#### 1.2 mysql配置

启动mysql

```
systemctl start mysqld.service
```

查看mysql是否启动成功

```
systemctl status mysqld.service
```

查找root默认密码

```
grep "password" /var/log/mysqld.log
```

默认生成的密码为：`E1!Irss%T,ev`

使用临时密码，进入mysql客户端，然后更改密码

```shell
 mysql -uroot -p
 set global validate_password_policy=LOW;
 set global validate_password_length=6;
 ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
 #开启mysql的远程连接权限
grant all privileges  on  *.* to 'root'@'%' identified by '123456' with grant option;
flush privileges;

```

### 2、Hive客户端安装配置

安装的hive客户端版本为hive-1.1.0-cdh5.14.2.tar.gz

1）解压到指定目录

```
tar -zxf hive-1.1.0-cdh5.14.2.tar.gz -C /ross/install/
```

2）配置conf的hive-env.sh

```
cd /ross/install/hive-1.1.0-cdh5.14.2/conf/

mv  hive-env.sh.template hive-env.sh

vim hive-env.sh
```

修改hive-env.sh文件：

```
#配置HADOOP_HOME路径
export HADOOP_HOME=/kkb/install/hadoop-2.6.0-cdh5.14.2/
#配置HIVE_CONF_DIR路径
export HIVE_CONF_DIR=/kkb/install/hive-1.1.0-cdh5.14.2/conf
```

3）配置hive-site.xml

需要创建一个hive-site.xml文件，并修改配置为：

```xml
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://node1:3306/hive?createDatabaseIfNotExist=true&amp;characterEncoding=latin1&amp;useSSL=false</value>
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
                <value>123456</value>
        </property>
        <property>
                <name>hive.cli.print.current.db</name>
                <value>true</value>
        </property>
        <property>
                <name>hive.cli.print.header</name>
            <value>true</value>
        </property>
        <property>
                <name>hive.server2.thrift.bind.host</name>
                <value>node1</value> 
        </property>
</configuration>
```

其中avax.jdo.option.ConnectionUR配置数据库的连接串，然后分别配置数据库的驱动、账号和密码；hive.server2.thrift.bind.host配置hiveserver2服务启动在哪一台机器。

4）配置日志地址

```
mkdir -p /ross/install/hive-1.1.0-cdh5.14.2/logs/
cd /ross/install/hive-1.1.0-cdh5.14.2/conf/
mv hive-log4j.properties.template hive-log4j.properties
vim hive-log4j.properties

#更改以下内容，设置我们的日志文件存放的路径
hive.log.dir=/kkb/install/hive-1.1.0-cdh5.14.2/logs/
```

ps: ==需要将mysql的驱动包上传到hive的lib目录下==

* 例如 mysql-connector-java-5.1.38.jar

### 3、Hive客户端交互

先启动hadoop集群和mysql服务==

#### 3.1 Hive交互shell

```
cd /ross/install/hive-1.1.0-cdh5.14.2
bin/hive
```

只能在安装机器上使用；

#### 3.2 Hive JDBC服务

* 启动hiveserver2服务

  * 前台启动

    ~~~shell
    cd /ross/install/hive-1.1.0-cdh5.14.2
    bin/hive --service hiveserver2
    ~~~

  * 后台启动

  ~~~shell
  cd /ross/install/hive-1.1.0-cdh5.14.2
  nohup  bin/hive --service hiveserver2  &
  ~~~

* beeline连接hiveserver2

  重新开启一个会话窗口，然后使用beeline连接hive

  ~~~shell
  cd /ross/install/hive-1.1.0-cdh5.14.2
  bin/beeline
  beeline> !connect jdbc:hive2://node03:10000
  ~~~

可以通过远程使用。在hive-site.xml中可以配置连接数

#### 3.3 Hive的命令

* hive  -e执行 sql语句
  * 使用 –e  参数来直接执行hql的语句

~~~
cd /kkb/install/hive-1.1.0-cdh5.14.2/
bin/hive -e "show databases"
~~~

* hive  -f执行 sql文件

  * 使用 –f  参数执行包含hql语句的文件

  * node03执行以下命令准备hive执行脚本

  * ```
    cd /ross/install/
    vim hive.sql
    
    文件内容如下
    create database if not exists myhive;
    
    通过以下命令来执行我们的hive脚本
    cd /ross/install/hive-1.1.0-cdh5.14.2/
    bin/hive -f /ross/install/hive.sql 
    ```

一般用于开发完成后上线使用

