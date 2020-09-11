## kubectl客户端工具

kubectl是最常用的客户端工具之一，它提供了基于命令行访问API的简洁方式，能够满足绝大多数的操作需求。如创建资源对象时候，kubectl会将JSON格式的清单以POST方式提交至API Server。

kubectl 命令的基本格式

```
kubectl [command] TYPE NAME flags
#command 为子命令如get,create,delete,apply,run,expose,edit,label等
#TYPE 为操作的资源类型，如pods、services、nodes等，区分大小写，支持简写格式
#NAME 对象名称，也可使用TYPE/NAME方式表示资源对象
#flags 命令行选项 如--port -o --server
```

- 详细的kubectl子命令列表如下表格，摘自P58

  |      命令      | 功能说明                                                     |
  | :------------: | :----------------------------------------------------------- |
  |     create     | 通过文件或标准输入创建资源                                   |
  |     expose     | 基于rc、service、deployment或pod创建service资源              |
  |      run       | 通过创建Deployment在集群中运行指定的镜像                     |
  |      set       | 设置指定的资源属性                                           |
  |      get       | 显示一个或多个资源                                           |
  |    explain     | 资源文件注解                                                 |
  |      edit      | 编辑资源(部分资源字段只读)                                   |
  |     delete     | 基于文件名、标准输入、资源或名字，以及资源和选择器删除资源   |
  |    rollout     | 部署命令：管理资源的滚动更新                                 |
  | rolling-update | 部署命令：对ReplicationController执行滚动更新                |
  |     scale      | 部署命令：伸缩Deployment、ReplicaSet、RC或Job规模            |
  |   autoscale    | 部署命令：对Deployment、ReplicaSet或RC进行自动伸缩           |
  |  certificate   | 集群管理命令：配置资源证书                                   |
  |  cluster-info  | 集群管理命令：打印集群信息                                   |
  |      top       | 集群管理命令：打印资源使用率(CPU/Memory/Storage)             |
  |     cordon     | 集群管理命令：将指定的node资源设置为不可用(unscheduable)状态 |
  |    uncordon    | 集群管理命令：将指定的node资源设置为可用状态(scheduable)状态 |
  |     drain      | 集群管理命令：”排干“指定node的负载以进入”维护“模式           |
  |     taint      | 集群管理命令：为node申明污点及标准行为                       |
  |    describe    | 排除调试命令：显示指定资源或资源组的详细信息                 |
  |      logs      | 排除调试命令：显示一个Pod内某容器的日志                      |
  |     attach     | 排除调试命令：附加终端至一个运行中的容器                     |
  |      exec      | 排除调试命令：在容器中执行指定命令                           |
  |  port-forward  | 排除调试命令：将本地的一个或多个端口转发至指定的容器         |
  |     proxy      | 排除调试命令：创建能够访问Kubernetes API Server的代理        |
  |       cp       | 排除调试命令：在容器间复制文件或目录                         |
  |      auth      | 排除调试命令：打印授权信息                                   |
  |     apply      | 高级命令：基于清单文件或stdin将配置应用于资源                |
  |     patch      | 高级命令：使用策略合并补丁更新字段资源                       |
  |    replace     | 高级命令：基于文件或stdin替换一个资源                        |
  |    convert     | 高级命令：为不同的API版本转换配置文件                        |
  |     label      | 设置命令：更新指定资源的标签                                 |
  |   annotation   | 设置命令：更新资源的详细信息(注解)                           |
  |   completion   | 设置命令：输出指定的shell的补全码                            |
  |    version     | 其他命令：打印服务端及客户端的版本信息                       |
  |  api-versions  | 其他命令：以“group/version” 格式打印服务器支持的API版本信息  |
  |     config     | 其他命令：配置kubeconfig文件内容                             |
  |     plugin     | 其他命令：运行命令行插件                                     |
  |      help      | 其他命令：打印指定命令的帮助信息                             |

- kubectl输出格式

  | 输出格式          | 格式说明                            |
  | ----------------- | ----------------------------------- |
  | -o wide           | 显示资源的额外信息                  |
  | -o name           | 仅打印资源的名称                    |
  | -o yaml           | YAML格式输出对象资源信息            |
  | -o json           | JSON格式输出API对象资源             |
  | -o go-template    | 以自定义的go模板格式输出API对象信息 |
  | -o custom-columns | 自定义输出的字段                    |

- 其他更多命令

  kubectl更多通用选项可以使用kubectl options 获取

  - -s 或 --server：指定API Server的地址和端口
  - --kubeconfig：使用的kubeconfig文件路径，默认为~/.kube/config
  - --namespace：命令执行的目标及空间