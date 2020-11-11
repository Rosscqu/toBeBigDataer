## CDH安装



### 1、环境准备

机器规划

| IP             | 作用名        | 账号/密码         | 环境配置                               | 安装 |
| -------------- | ------------- | ----------------- | -------------------------------------- | ---- |
| 192.168.xx.100 | manager/node1 | hadoop/bigdata002 | 关闭防火墙和SELinux,host映射，时钟同步 | JDK8 |
| 192.168.xx.110 | node2         | hadoop/bigdata002 | 关闭防火墙和SELinux,host映射，时钟同步 | JDK8 |
| 192.168.xx.120 | node3         | hadoop/bigdata002 | 关闭防火墙和SELinux,host映射，时钟同步 | JDK8 |



集群配置

| 主机  | mysql |      |
| ----- | ----- | ---- |
| node1 | Y     |      |
| node2 |       |      |
| node3 |       |      |



执行命令

```
yum install -y chkconfig python bind-utils psmisc libxsltzlib sqlite cyrus-sasl-plain  cyrus-sasl-gssapi fuse fuse-libs redhat-lsb
```



### 2、manager节点安装



#### 2.1 mysql安装



```
create database cmserver default charset utf8 collate utf8_general_ci;
grant all on cmserver.* to 'cmserveruser'@'%' identified by '123456';

create database metastore default charset utf8 collate utf8_general_ci;
grant all on metastore.* to 'hiveuser'@'%' identified by '123456';

create database amon default charset utf8 collate utf8_general_ci;
grant all on amon.* to 'amonuser'@'%' identified by '123456';

create database rman default charset utf8 collate utf8_general_ci;
grant all on rman.* to 'rmanuser'@'%' identified by '123456';

create database oozie default charset utf8 collate utf8_general_ci;
grant all on oozie.* to 'oozieuser'@'%' identified by '123456';

create database hue default charset utf8 collate utf8_general_ci;
grant all on hue.* to 'hueuser'@'%' identified by '123456';
```



#### 2.2 安装Httpd服务

```
yum install httpd
systemctl start httpd
systemctl enable httpd.service 设置httpd服务开机自启
```



#### 2.3 导入GPG key

```
rpm --import RPM-GPG-KEY-cloudera
```



#### 2.4 安装Cloudera-Manager

```
rpm -ivh cloudera-manager-daemons-6.2.0-968826.el7.x86_64.rpm
rpm -ivh cloudera-manager-agent-6.2.0-968826.el7.x86_64.rpm
rpm -ivh cloudera-manager-server-6.2.0-968826.el7.x86_64.rpm
```



### 4、安装Cloudera Manager Server和Agent



