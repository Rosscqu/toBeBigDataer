## Flink架构

### 1、Flink集群基础架构





#### 1.1 JobManager

JobManager是管理节点，每个集群至少一个，管理整个集群的计算资源、Job的管理和调度执行、checkpoint协调等。



#### 1.2 TaskManager

TaskManager是计算节点，负责计算资源的提供。





#### 1.3 Client

Client是客户端，负责执行应用的main()方法、解析jobGraph对象，并最终将JobGraph提交到JobManager运行，同时监控job执行的状态。



### 2、Flink的集群部署模式

根据以下两个条件：

1）集群的生命周期和资源隔离；

2）根据程序main()方法执行在Client还是JobManager;

将集群部署模式分为三个类型：

1）Session模式：共享资源，运行在Client上

共享JobManager和TaskManager，所有提交的代码都在一个Runtime中运行；





优点：

- 资源充分共享，提升资源的利用率；
- Job在Flink Session集群中管理，运维简单；

缺点：

- 资源隔离相对较差；
- 非Native类型部署，TM不易扩展，Slot计算资源伸缩性较差。



2）Per-job 模式：不共享资源，运行在Client上

独享JobManager和TaskManager，好比为每个Job单独启动一个RunTime；





优点：

- Job与Job之间资源隔离充分；
- 资源根据Job需要进行申请，TM Slots数量可以不同；

缺点：

- 资源相对比较浪费，JobManager需要消耗资源；
- Job管理完全交给ClusterManager，管理复杂；



3）Application模式：不共享资源，运行在Cluster上

- Application的main()运行在Cluster中，而不是客户端上；
- 每个Application对应一个Runtime，Application可以包含多个job。





优点：

- 有效降低带宽和客户端负载；
- Application实现资源隔离，Application中实现资源共享；

缺点：

- 功能太新，还未经过生产验证；
- 仅支持Yarn和K8S。



### 3、Flink集群部署

Flink支持的资源管理器部署集群：

- Standalone
- Yarn
- Mesos
- Docker
- Kubernetes



#### 3.1 Flink Standalone部署与原理





#### 3.2 Flink on Yarn部署与原理

