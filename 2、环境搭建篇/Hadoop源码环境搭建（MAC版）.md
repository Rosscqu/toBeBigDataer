## Hadoop源码环境搭建（MAC版）



### 1、原材料准备

JDK版本：1.8

maven：3.6.3

Protobuf：用于RPC系统和持续数据存储系统，是一种轻便高效的可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。版本使用2.5.0

Hadoop版本：2.8.5

IDE工具：IDEA



### 2、protobuf安装

```shell
tar -zxvf protobuf-2.5.0.tar.gz
cd protobuf-2.5.0
./configure 
make
make check
make install
```

验证是否安装成功：

```shell
protoc --version
```



### 3、Hadoop编译

```
tar -zxvf hadoop-2.8.5-src.tar.gz
cd hadoop-2.8.5-src
mvn package -Pdist,native -DskipTests-Dtar-Dmaven.javadoc.skip=true
```

编译Hadoop遇到的坑：

1）Failed to execute goal org.apache.maven.plugins:maven-javadoc-plugin:2.8.1:jar **(module-javadocs)** on project hadoop-yarn-server-common

<img src="../../../Library/Application Support/typora-user-images/image-20200822193331101.png" alt="image-20200822193331101" style="zoom:50%;" />



解决方式：

- JDK8版本过高，换成JDK7；
- 换成命令行`*mvn package -Pdist,native -DskipTests-Dtar*-Dmaven.javadoc.skip=true`



2）Failed to execute goal org.apache.hadoop:hadoop-maven-plugins:2.10.0:cmake-compile **(cmake-compile)** on project hadoop-common: **Error executing CMake**

安装Cmake

```
brew install cmake
```



3）Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.21.0:test **(default-test)** on project hadoop-common: **There are test failures**

<img src="../../../Library/Application Support/typora-user-images/image-20200822202340060.png" alt="image-20200822202340060" style="zoom:50%;" />

```
mvn package -Pdist,native -DskipTests -Dtar -Dmaven.javadoc.skip=true

mvn clean package -Pdist,native -DskipTests-Dtar-Dmaven.javadoc.skip=true -Dmaven.test.skip=true
```

