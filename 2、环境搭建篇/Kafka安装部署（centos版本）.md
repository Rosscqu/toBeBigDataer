## Kafka安装部署（centos版本）



### 1、Kafka集群安装

- 1、下载安装包（http://kafka.apache.org）

  ~~~
  kafka_2.11-1.0.1.tgz
  ~~~

- 2、规划安装目录

  ~~~
  /ross/install
  ~~~

- 3、上传安装包到服务器中

  ~~~
  通过FTP工具上传安装包到node01服务器上
  ~~~

- 4、解压安装包到指定规划目录

  ~~~shell
  tar -zxvf kafka_2.11-1.0.1.tgz -C /ross/install
  ~~~

- 5、重命名解压目录

  ~~~shell
  mv kafka_2.11-1.0.1 kafka
  ~~~

- 6、修改配置文件

  - 在node01上修改

    - 进入到kafka安装目录下有一个config目录

      - vi server.properties

        ```shell
        #指定kafka对应的broker id ，唯一
        broker.id=0
        #指定数据存放的目录
        log.dirs=/ross/install/kafka/kafka-logs
        #指定zk地址
        zookeeper.connect=node01:2181,node02:2181,node03:2181
        #指定是否可以删除topic ,默认是false 表示不可以删除
        delete.topic.enable=true
        #指定broker主机名
        host.name=node01
        ```

    - 配置kafka环境变量

      - vi /etc/profile

        ```
        export KAFKA_HOME=/ross/install/kafka
        export PATH=$PATH:$KAFKA_HOME/bin
        ```

- 6、分发kafka安装目录到其他节点

  ```
  scp -r kafka node02:/ross/install
  scp -r kafka node03:/ross/install
  scp /etc/profile node02:/etc
  scp /etc/profile node03:/etc
  ```

- 7、修改node02和node03上的配置

  - node02

    - vi server.properties

      ```shell
      #指定kafka对应的broker id ，唯一
      broker.id=1
      #指定数据存放的目录
      log.dirs=/ross/install/kafka/kafka-logs
      #指定zk地址
      zookeeper.connect=node01:2181,node02:2181,node03:2181
      #指定是否可以删除topic ,默认是false 表示不可以删除
      delete.topic.enable=true
      #指定broker主机名
      host.name=node02
      ```

  - node03

    - vi server.properties

      ```shell
      #指定kafka对应的broker id ，唯一
      broker.id=2
      #指定数据存放的目录
      log.dirs=/ross/install/kafka/kafka-logs
      #指定zk地址
      zookeeper.connect=node01:2181,node02:2181,node03:2181
      #指定是否可以删除topic ,默认是false 表示不可以删除
      delete.topic.enable=true
      #指定broker主机名
      host.name=node03
      ```

### 2. kafka集群启动和停止

#### 2.1 启动

- 先启动zk集群

- 然后在所有节点执行脚本

  ```shell
  nohup kafka-server-start.sh /ross/install/kafka/config/server.properties >/dev/null 2>&1 &
  ```

- 一键启动kafka

  - start_kafka.sh

    ```shell
    #!/bin/sh
    for host in node01 node02 node03
    do
            ssh $host "source /etc/profile;nohup kafka-server-start.sh /ross/install/kafka/config/server.properties >/dev/null 2>&1 &" 
            echo "$host kafka is running"
    
    done
    ```

    

#### 2.2  停止

- 所有节点执行关闭kafka脚本

  ```
  kafka-server-stop.sh
  ```

- 一键停止kafka

  - stop_kafka.sh

    ```shell
    #!/bin/sh
    for host in node01 node02 node03
    do
      ssh $host "source /etc/profile;nohup /ross/install/kafka/bin/kafka-server-stop.sh &" 
      echo "$host kafka is stopping"
    done
    ```
