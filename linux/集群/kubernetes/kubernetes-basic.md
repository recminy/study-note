# kubernetes

kubernetes就是一个运行容器的集群，简称k8s,它将所有节点上的资源整合到一个强大的虚拟资源池里，代替单一的服务器

## master

好比一个操作系统内核，主要职责抽象资源并调度任务

## Pod

就像运行于用户空间的进程，为K8s最小调度单元，用于运行容器，一个Pod中可运行多个容器，但一般一个Pod中仅运行一个容器

## APIServer

负责接收并相应客户端提交的任务接口

## Pod控制器

- ReplicaSet：新一代的ReplicationController,代用户创建指定数量的应用副本，并确保副本数量始终处于用户所指定的数量，多还少补，还支持扩缩容
- Deployment：工作在ReplicaSet上，支持扩缩容，滚动更新，声明式配置（可以在应用运行时动态修改配置）,Deployment是帮我们管理无状态应用的最佳选择，且它也只能管理无状态应用
- DaemonSet:确保集群的每个节点只运行一个特定的Pod副本，通用用于实现一些系统级别的后台任务，如日志收集服务
- Job/CronJob:执行系统及的作业，Job只能执行一次，CronJob为周期性作业
- StatefulSet：管理有状态的应用，每一个应用（Pod副本）均被单独管理，它拥有自己的独有的标识及数据集

## Service

service 是微服务的一种实现，实际上是一种抽象，通过规则定义出由多个Pod对象组合而成的逻辑集合，并附带访问这组Pod对象的策略（Strategy）,其主要类型有三种

- ClusterIP:仅用于集群内部通信
- NodePort：接入集群外部的流量，他工作与每个节点的主机IP上
- LoadBalance：它可以把外部请求负载均衡至多个Node的主机IP的NodePort上

每一种访问方式都以其前一种为基础才能实现，而LoadBalance需要协同集群外部的组件才能实现，且外部组件不受k8s管理



