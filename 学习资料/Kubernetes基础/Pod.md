### Pod概念

#### 什么是Pod？
Pod可简单地理解为是一组、一个或多个容器，具有共享存储/网络及如何运行容器的规范。Pad包含一个或多个相对紧密耦合的应用程序容器，处于同一个Pod中的容器共享同样的存储空间（Volume，卷或存储卷）、IP地址和Port端口，容器之间使用localhost:port相互访问。根据Docker的构造，Pod可被建模为一组具有共享命令空间、卷、IP地址和Port端口的Docker容器。
Pod包含的容器最好是一个容器只运行一个进程。每个Pod包含一个pause容器，pause容器是Pod的父容器，它主要负责僵尸进程的回收管理。
Kubernetes为每个Pod都分配一个唯一的IP地址，这样就可以保证应用程序使用同一端口，避免了发生冲突的问题。


#### Pod字段解析
````
apiVersion: v1 # 必选，API的版本号
kind: Pod	# 必选，类型Pod
metadata:	# 必选，元数据
  name: nginx	# 必选，符合RFC 1035规范的Pod名称
  namespace: web-testing # 可选，不指定默认为default，Pod所在的命名空间
  labels:	# 可选，标签选择器，一般用于Selector
    - app: nginx
  annotations:	# 可选，注释列表
    - app: nginx
spec:	# 必选，用于定义容器的详细信息
  containers:	# 必选，容器列表
  - name: nginx	# 必选，符合RFC 1035规范的容器名称
    image: nginx: v1	# 必选，容器所用的镜像的地址
    imagePullPolicy: Always	# 可选，镜像拉取策略
	command: 
	- nginx	# 可选，容器启动执行的命令
	- -g
	- "daemon off;"
		workingDir: /usr/share/nginx/html	# 可选，容器的工作目录
		volumeMounts:	# 可选，存储卷配置
		- name: webroot # 存储卷名称
		  mountPath: /usr/share/nginx/html # 挂载目录
		  readOnly: true	# 只读
		ports:	# 可选，容器需要暴露的端口号列表
		- name: http	# 端口名称
		  containerPort: 80	# 端口号
		  protocol: TCP	# 端口协议，默认TCP
		env:	# 可选，环境变量配置
		- name: TZ	# 变量名
		  value: Asia/Shanghai
		- name: LANG
		  value: en_US.utf8
		resources:	# 可选，资源限制和资源请求限制
		  limits:	# 最大限制设置
			cpu: 1000m
			memory: 1024MiB
		  requests:	# 启动所需的资源
			cpu: 100m
			memory: 512MiB
		readinessProbe:	# 可选，容器状态检查
		httpGet:	# 检测方式
			path: /	# 检查路径
			port: 80	# 监控端口
		  timeoutSeconds: 2	# 超时时间 
		  initialDelaySeconds: 60	# 初始化时间
		livenessProbe:	# 可选，监控状态检查
		  exec:	# 检测方式
			command: 
			- cat
			- /health
		  httpGet:	# 检测方式
			path: /_health
			port: 8080
			httpHeaders:
			- name: end-user
			  value: jason
		  tcpSocket:	# 检测方式
			port: 80
		  initialDelaySeconds: 60	# 初始化时间
		  timeoutSeconds: 2	# 超时时间
		  periodSeconds: 5	# 检测间隔
		  successThreshold: 2 # 检查成功为2次表示就绪
		  failureThreshold: 1 # 检测失败1次表示未就绪
		securityContext:	# 可选，限制容器不可信的行为
		  provoleged: false
	  restartPolicy: Always	# 可选，默认为Always
	  nodeSelector:	# 可选，指定Node节点
		region: subnet7
	  imagePullSecrets:	# 可选，拉取镜像使用的secret
	  - name: default-dockercfg-86258
	  hostNetwork: false	# 可选，是否为主机模式，如是，会占用主机端口
	  volumes:	# 共享存储卷列表
	  - name: webroot # 名称，与上述对应
		emptyDir: {}	# 共享卷类型，空
		hostPath:		# 共享卷类型，本机目录
		  path: /etc/hosts
		secret:	# 共享卷类型，secret模式，一般用于密码
		  secretName: default-token-tf2jp # 名称
		  defaultMode: 420 # 权限
		  configMap:	# 一般用于配置文件
		  name: nginx-conf
		  defaultMode: 420

````