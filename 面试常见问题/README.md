[toc]

# 面试不要不懂装懂，不会就是不会，不可能每个人都接触过所有的知识！

## 1. 基础问题

### 1.1 Service是怎么关联Pod的？（课程Service章节）
答：创建Pod是都会定义Pod的便签，比如role=frontend，Service通过Selector字段匹配该标签即可关联至该Pod，Pod和Service需要在同一个namespace，[中文文档](https://kubernetes.io/zh/docs/concepts/services-networking/connect-applications-service/)。

### 1.2 HPA V1 V2的区别
答：HPA v1为稳定版自动水平伸缩，只支持CPU指标。V2为beta版本，分为v2beta1(支持CPU、内存和自定义指标)，v2beta2(支持CPU、内存、自定义指标Custom和额外指标ExternalMetrics)，从k8s 1.11之后，度量指标的采集依赖metrics-server，弃用了heapster，[中文文档](https://kubernetes.io/zh/docs/tasks/run-application/horizontal-pod-autoscale/)。

### 1.3 Pod生命周期（课程Pod章节）
````
答：
   Pod创建：
      1. API Server 在接收到创建pod的请求之后，会根据用户提交的参数值来创建一个运行时的pod对象。
      2. 根据 API Server 请求的上下文的元数据来验证两者的 namespace 是否匹配，如果不匹配则创建失败。
      3. Namespace 匹配成功之后，会向 pod 对象注入一些系统数据，如果 pod 未提供 pod 的名字，则 API Server 会将 pod 的 uid 作为 pod 的名字。
      4. API Server 接下来会检查 pod 对象的必需字段是否为空，如果为空，创建失败。
      5. 上述准备工作完成之后会将在 etcd 中持久化这个对象，将异步调用返回结果封装成 restful.response，完成结果反馈。
      6. API Server 创建过程完成，剩下的由 scheduler 和 kubelet 来完成，此时 pod 处于 pending 状态。
      7. Scheduler选择出最优节点。
      8. Kubelet启动该Pod。
   Pod删除：
      1. 用户发出删除 pod 命令
      2. 将 pod 标记为“Terminating”状态
         监控到 pod 对象为“Terminating”状态的同时启动 pod 关闭过程
         endpoints 控制器监控到 pod 对象关闭，将pod与service匹配的 endpoints 列表中删除
         Pod执行PreStop定义的内容
      3. 宽限期（默认30秒）结束之后，若存在任何一个运行的进程，pod 会收到 SIGKILL 信号
      4. Kubelet 请求 API Server 将此 Pod 资源宽限期设置为0从而完成删除操作
````

### 1.4 Kubernetes Master节点高可用（课程Master节点和Node节点章节）
答：Kube-APIServer为无状态服务，可以启动多个，通过负载均衡进行轮训。ControllerManager和Scheduler为有状态服务，多节点启动会进行选主，主节点信息保存在kube-system命名空间下的对应名称的endpoint中

### 1.5 QoS（课程QoS章节）
答： 最高级别：Guaranteed节点资源不够时第一个被杀掉， Burstable第二个被杀掉，BestEffort第一个被杀掉

### 1.6 flannel和calico（课程安装章节）
答：如果没有用过flannel可以直接说没有用过flannel，都是用的calico，因为calico性能强大，并且配置简单。Flannel的host-gw虽然性能好，但是只能用于大二层网络，vxlan对内核要求高，并且flannel不支持网络策略，所以采用calico。因为公司和公有云网络环境不支持BGP，所以目前采用的都是IPIP模式。

### 1.7 Helm优点（课程Helm章节）
答：大型项目更加方便管理，可以一键创建一个环境，可以对整个项目进行版本升级、回滚，部署更加方便。

### 1.8 公司的架构是什么样的？
答：我们的架构是这样的，三台master，三台etcd，etcd和master没有放在一起。然后在指定的节点上部署了ingress nginx，然后外部有个网关（可以选择性说网关是硬件设备F5或者DMZ的nginx，或者公有云的LB）连接到了k8s ingress节点的80和433，然后有个通配符域名指向了ingress，在ingress上面又做的分发。


## 2. 日志监控
### 2.1 容器内日志怎么采集的？（课程日志采集章节）
答：容器内日志我们是使用filebeat进行采集的，filebeat以sidecar的形式和业务应用运行在同一个Pod内，使用emptyDir进行日志文件的共享。

### 2.2 Fluentd
答：Fluentd配置简单，并且Docker日志一般是json输出，使用fluentd收集更加方便，当然filebeat也是可以采集节点日志的。

### 2.3 日志的索引（课程日志采集章节）
答：为了更快的查询日志，一般我们会根据集群、命名空间、资源名称进行添加索引。

### 2.4 etcd怎么监控的？（课程自带metrics接口应用的监控）
答：etcd属于云原生应用，自带了metrics接口，可以直接请求metrics接口即可获取到监控数据，一般监控etcd的状态、leader是否正常、选择次数、选主失败次数、集群延迟、落盘延迟等。（此问题可以根据监控项自行补充）

### 2.5 黑盒监控blackbox（课程黑盒监控）
答：黑盒监控可以监控http、tcp的监控状态、延迟、解析速度、证书到期时间等指标，可以根据课程的监控图自行补充。

### 2.6 状态码监控
答：可以这么回答，我们使用的是ingress，ingress也是用Prometheus监控的，可以监控到某个应用的请求状态，比如多个200、502、403等，课程ingress监控章节。

## 3. 存储问题
### 3.1 Rook问题（课程rook章节）
答：Rook现在已经毕业了，之前虽然没有毕业，但是对ceph的支持已经是stable了，并且rook降低了ceph的学习成本，几乎不用运维，所以我们采用了Rook。使用Rook操作ceph扩容也是非常简单的，只需要更改rook创建ceph集群的资源文件即可。

### 3.2 如何对接外部CEPH（课程volume和动态存储章节）
答：对接的方式有很多，使用Rook可以对接外部ceph，使用volume、pvc和storageClass都可以对接外部ceph。


以上问答只是个人见解，不一定是最好的回答，大家可以自行查阅网上资料。
