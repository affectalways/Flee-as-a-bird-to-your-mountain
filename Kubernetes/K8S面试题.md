# K8S面试题

### 目录

- [k8s是什么](#k8s是什么)
- [k8s相关概念](#k8s相关概念)
  - [Container](#Container)
  - [Pod](#Pod)
  - [Node](#Node)
  - [Namespace](#Namespace)
  - [Service](#Service)
  - [Label](#Label)
- [K8s架构的组成是什么](#K8s架构的组成是什么)
- [K8S架构细分](#K8S架构细分)
- [k8s的健康检查机制是什么](#k8s的健康检查机制是什么)
- [镜像的下载策略有哪些](#镜像的下载策略有哪些)
- [Pod 的状态有哪些](#Pod 的状态有哪些)
- [k8s创建一个pod流程](#k8s创建一个pod流程)
- [删除pod流程](#删除pod流程)
- [service是什么](#service是什么)
- [怎么服务注册](#怎么服务注册)
- [持久化方式](#持久化方式)
- [常用命令](#常用命令)

### k8s是什么

Kubernetes是一个针对容器应用，进行自动部署、弹性伸缩和管理的开源系统，主要功能是生产环境中的容器编排。

</br></br>

### K8s架构的组成是什么

和大多数分布式系统一样，K8S集群至少需要一个主节点（Master）和多个工作节点（Node）。

- 主节点主要用于暴露API，调度部署和节点的管理；
- 工作节点Node是Kubernetes集群架构中运行Pod的服务节点，是Kubernetes集群操作的单元，是Pod运行的宿主机。运行Docker Eninge服务，守护进程kubelet及负载均衡器kube-proxy。

</br></br>

### k8s架构图及交互流程

![](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/K8S%E9%9D%A2%E8%AF%951.png)

k8s 主要有 Master 节点和工作节点组成。主节点主要对集群做出全局决策(比如调度)，以及检测和响应集群事件(例如资源不足，自动扩缩容)；从节点负责维护运行的 Pod 并进行通信的网络代理。

**Master 节点（默认不参加实际工作）**主要有以下组件：

- kube-apiserver：**负责对外暴露 Kubernetes API**，在K8s架构中承担的是“桥梁”的角色，作为资源操作的唯一入口，它提供了认证、授权、访问控制、API注册和发现等机制。**客户端与k8s群集及K8s内部组件的通信，都要通过Api Server这个组件**
- etcd：作为保存 Kubernetes 所有集群数据的后台数据库，保存整个集群的状态
- kube-scheduler：负责资源的调度，按照预定的调度策略将pod调度到相应的node节点上
- kube-controller-manager：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等
- Kubectl：客户端命令行工具，作为整个K8s集群的操作入口，可以使用该工具控制Kubernetes集群管理器，如检查群集资源，创建、删除和更新组件，查看应用程序。

**Node 节点**有以下组件：

- kubelet：主要负责执行、监控由调度器分配的 Pod，相当于是 Master 在每个 Node 节点上的代理。一般运行在所有的节点，是Node节点的代理，当Scheduler确定某个node上运行pod之后，会将pod的具体信息（image，volume）等发送给该节点的kubelet，kubelet根据这些信息创建和运行容器，并向master返回运行状态。（自动修复功能：如果某个节点中的容器宕机，它会尝试重启该容器，若重启无效，则会将该pod杀死，然后重新创建一个容器）
- kube-proxy：k8s 在每个节点上的网络代理，负责为 Service 提供集群内部的服务发现和负载均衡。
- Pod：是k8s集群里面最小的单位。每个pod里边可以运行一个或多个container（容器），如果一个pod中有两个container，那么container的USR（用户）、MNT（挂载点）、PID（进程号）是相互隔离的，UTS（主机名和域名）、IPC（消息队列）、NET（网络栈）是相互共享的。我比较喜欢把pod来当做豌豆夹，而豌豆就是pod中的container；
- container-runtime：是负责管理运行容器的软件，比如docker

</br></br>

### k8s相关基础概念

#### Master

Kubernetes集群的管理节点，负责管理集群，提供集群的资源数据访问入口。拥有etcd存储服务（可选），运行Api Server进程，Controller Manager服务进程及Scheduler服务进程。

</br>

#### Node（worker）

Node 是 Pod 真正运行的主机，可以是物理机，也可以是虚拟机。

Node（worker）是Kubernetes集群架构中运行Pod的服务节点，是Kubernetes集群操作的单元，用来承载被分配Pod的运行，是Pod运行的宿主机。运行Docker Eninge服务，守护进程kubelet及负载均衡器kube-proxy。

为了管理 Pod，每个 Node 节点上至少要运行 container runtime（比如 docker 或者 rkt）、kubelet 和 kube-proxy 服务。

</br>

#### Pod

k8s 使用 Pod 来管理容器，一个 Pod 可以包含一个或多个容器。它是一组紧密关联的容器集合，共享 PID、IPC、Network 和 UTS namespace，是 Kubernetes 调度的基本单位。

Pod 内的多个容器共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

</br>

#### Deployment

Deployment在内部使用了RS来实现目的，Deployment相当于RC的一次升级，其最大的特色为可以随时获知当前Pod的部署进度。

</br>

#### Container

即容器，通过镜像包含软件的运行环境，加上 namespace 的隔离功能，使得容器可以很方便的在任何地方运行。

</br>

#### Namespace

Namespace用于实现多租户的资源隔离，可将集群内部的资源对象分配到不同的Namespace中，形成逻辑上的不同项目、小组或用户组，便于不同的Namespace在共享使用整个集群的资源的同时还能被分别管理。

</br>

#### Service

Service 是应用服务的抽象，通过 labels 为应用提供负载均衡和服务发现。匹配 labels 的 Pod IP 和端口列表组成 endpoints，由 kube-proxy 负责将服务 IP 负载均衡到这些 endpoints 上。

每个 Service 都会自动分配一个 cluster IP（仅在集群内部可访问的虚拟地址）和 DNS 名，其他容器可以通过该地址或 DNS 来访问服务，而不需要了解后端容器的运行。

</br>

#### Label

Label 是识别 Kubernetes 对象的标签，以 key/value 的方式附加到对象上。Label 不提供唯一性，并且实际上经常是很多对象（如 Pods）都使用相同的 label 来标志具体的应用。

</br>

#### Replication Controller

Replication Controller用来管理Pod的副本，保证集群中存在指定数量的Pod副本。集群中副本的数量大于指定数量，则会停止指定数量之外的多余容器数量。反之，则会启动少于指定数量个数的容器，保证数量不变。Replication Controller是实现弹性伸缩、动态扩容和滚动升级的核心。

</br>

#### Volume

Volume是Pod中能够被多个容器访问的共享目录，Kubernetes中的Volume是定义在Pod上，可以被一个或多个Pod中的容器挂载到某个目录下。

</br>

#### HPA（Horizontal Pod Autoscaler）

Pod的横向自动扩容，也是Kubernetes的一种资源，通过追踪分析RC控制的所有Pod目标的负载变化情况，来确定是否需要针对性的调整Pod副本数量。

</br></br>





### Kubernetes RC的机制

Replication Controller用来管理Pod的副本，保证集群中存在指定数量的Pod副本。当定义了RC并提交至Kubernetes集群中之后，Master节点上的Controller Manager组件获悉，并同时巡检系统中当前存活的目标Pod，并确保目标Pod实例的数量刚好等于此RC的期望值，若存在过多的Pod副本在运行，系统会停止一些Pod，反之则自动创建一些Pod。

</br></br>

### Kubernetes Replica Set和Replication Controller之间有什么区别

Replica Set和Replication Controller类似，都是确保在任何给定时间运行指定数量的Pod副本。不同之处在于RS使用基于集合的选择器，而Replication Controller使用基于权限的选择器。
</br></br>



### 简述Kubernetes和Docker的关系

Docker提供容器的生命周期管理和Docker镜像构建运行时容器。它的主要优点是将将软件/应用程序运行所需的设置和依赖项打包到一个容器中，从而实现了可移植性等优点。

Kubernetes用于关联和编排在多个主机上运行的容器。

</br></br>



### Kubernetes的优势、适应场景及其特点

Kubernetes作为一个完备的分布式系统支撑平台，其主要优势：

- 容器编排
- 轻量级
- 开源
- 弹性伸缩
- 负载均衡


Kubernetes常见场景：

- 快速部署应用
- 快速扩展应用
- 无缝对接新的应用功能
- 节省资源，优化硬件资源的使用


Kubernetes相关特点：

- 可移植：支持公有云、私有云、混合云、多重云（multi-cloud）。
- 可扩展: 模块化,、插件化、可挂载、可组合。
- 自动化: 自动部署、自动重启、自动复制、自动伸缩/扩展。

</br></br>

### Kubernetes的缺点或当前的不足之处

Kubernetes当前存在的缺点（不足）如下：

- 安装过程和配置相对困难复杂。
- 管理服务相对繁琐。
- 运行和编译需要很多时间。
- 它比其他替代品更昂贵。
- 对于简单的应用程序来说，可能不需要涉及Kubernetes即可满足。

</br></br>

### k8s的健康检查机制是什么

K8s中对于pod资源对象的健康状态检测，提供了三类probe（探针）来执行对pod的健康监测：

- ##### livenessProbe探针(存活探针)

可以根据用户自定义规则来判定pod是否健康，如果livenessProbe探针探测到容器不健康，则kubelet会根据其重启策略来决定是否重启，如果一个容器不包含livenessProbe探针，则kubelet会认为容器的livenessProbe探针的返回值永远成功。

- ##### ReadinessProbe探针(就绪探针)

有时候，应用程序会暂时性的不能提供通信服务（例如启动加载大文件）。在这种情况下，既不想杀死应用程序，也不想给它发送请求。Kubernetes 提供了就绪探测器来发现并缓解这些情况，设置后，流量将不会打到 Service 上。

- ##### startupProbe探针

启动检查机制，应用一些启动缓慢的业务，避免业务长时间启动而被上面两类探针kill掉，这个问题也可以换另一种方式解决，就是定义上面两类探针机制时，初始化时间定义的长一些即可。

每种探测方法能支持以下几个相同的检查参数，用于设置控制检查时间：

- initialDelaySeconds：初始第一次探测间隔，用于应用启动的时间，防止应用还没启动而健康检查失败
- periodSeconds：检查间隔，多久执行probe检查，默认为10s；
- timeoutSeconds：检查超时时长，探测应用timeout后为失败；
- successThreshold：成功探测阈值，表示探测多少次为健康正常，默认探测1次。

上面两种探针都支持以下三种探测方法：

**1）Exec：**通过执行命令的方式来检查服务是否正常，比如使用cat命令查看pod中的某个重要配置文件是否存在，若存在，则表示pod健康。反之异常。

Exec探测方式的yaml文件语法如下：

```
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:         #选择livenessProbe的探测机制
      exec:                      #执行以下命令
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5          #在容器运行五秒后开始探测
      periodSeconds: 5               #每次探测的时间间隔为5秒
```

在上面的配置文件中，探测机制为在容器运行5秒后，每隔五秒探测一次，如果cat命令返回的值为“0”，则表示健康，如果为非0，则表示异常。

**2）Httpget：**通过发送http/htps请求检查服务是否正常，返回的状态码为200-399则表示容器健康（注http get类似于命令curl -I）。

Httpget探测方式的yaml文件语法如下：

```
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    livenessProbe:              #采用livenessProbe机制探测
      httpGet:                  #采用httpget的方式
    scheme:HTTP         #指定协议，也支持https
        path: /healthz          #检测是否可以访问到网页根目录下的healthz网页文件
        port: 8080              #监听端口是8080
      initialDelaySeconds: 3     #容器运行3秒后开始探测
      periodSeconds: 3                #探测频率为3秒
```

上述配置文件中，探测方式为项容器发送HTTP GET请求，请求的是8080端口下的healthz文件，返回任何大于或等于200且小于400的状态码表示成功。任何其他代码表示异常。

**3）tcpSocket：** 通过容器的IP和Port执行TCP检查，如果能够建立TCP连接，则表明容器健康，这种方式与HTTPget的探测机制有些类似，tcpsocket健康检查适用于TCP业务。

tcpSocket探测方式的yaml文件语法如下：

```
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
- containerPort: 8080
#这里两种探测机制都用上了，都是为了和容器的8080端口建立TCP连接
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

在上述的yaml配置文件中，两类探针都使用了，在容器启动5秒后，kubelet将发送第一个readinessProbe探针，这将连接容器的8080端口，如果探测成功，则该pod为健康，十秒后，kubelet将进行第二次连接。

除了readinessProbe探针外，在容器启动15秒后，kubelet将发送第一个livenessProbe探针，仍然尝试连接容器的8080端口，如果连接失败，则重启容器。

探针探测的结果无外乎以下三者之一：

- Success：Container通过了检查；
- Failure：Container没有通过检查；
- Unknown：没有执行检查，因此不采取任何措施（通常是我们没有定义探针检测，默认为成功）。

若觉得上面还不够透彻，可以移步其官网文档：

https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

</br></br>

### Pod的LivenessProbe探针的常见方式

kubelet定期执行LivenessProbe探针来诊断容器的健康状态，通常有以下三种方式：

- ExecAction：在容器内执行一个命令，若返回码为0，则表明容器健康。
- TCPSocketAction：通过容器的IP地址和端口号执行TCP检查，若能建立TCP连接，则表明容器健康。
- HTTPGetAction：通过容器的IP地址、端口号及路径调用HTTP Get方法，若响应的状态码大于等于200且小于400，则表明容器健康。

</br></br>



### 镜像的下载策略有哪些

可通过命令`kubectl explain pod.spec.containers`来查看imagePullPolicy这行的解释。

K8s的镜像下载策略有三种：`Always、Never、IFNotPresent`；

- Always：镜像标签为latest时，总是从指定的仓库中获取镜像；
- Never：禁止从仓库中下载镜像，也就是说只能使用本地镜像；
- IfNotPresent：仅当本地没有对应镜像时，才从目标仓库中下载。

默认的镜像下载策略是：当镜像标签是latest时，默认策略是Always；当镜像标签是自定义时（也就是标签不是latest），那么默认策略是IfNotPresent。

</br></br>

### Pod 的状态有哪些

- Pending：API Server已经创建该Pod，且Pod内还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程。
- Running：pod 已正常创建，并且至少有一个容器正在运行。
- Succeeded：所有容器已成功启动运行。
- Failed：pod 的容器非正常退出。
- Unknown：无法获取 pod 状态，可能节点间通信出现问题。

</br></br>

### Kubernetes中Pod的重启策略

Pod重启策略（RestartPolicy）应用于Pod内的所有容器，并且仅在Pod所处的Node上由kubelet进行判断和重启操作。当某个容器异常退出或者健康检查失败时，kubelet将根据RestartPolicy的设置来进行相应操作。

> 可以通过命令“kubectl explain pod.spec”查看pod的重启策略。（restartPolicy字段）

Pod的重启策略包括Always、OnFailure和Never，默认值为Always。

- Always：当容器失效时，由kubelet自动重启该容器；
- OnFailure：当容器终止运行且退出码不为0时，由kubelet自动重启该容器；
- Never：不论容器运行状态如何，kubelet都不会重启该容器。


同时Pod的重启策略与控制方式关联，当前可用于管理Pod的控制器包括Replication Controller、Job、DaemonSet及直接管理kubelet管理（静态Pod）。

不同控制器的重启策略限制如下：

- RC和DaemonSet：必须设置为Always，需要保证该容器持续运行；
- Job：OnFailure或Never，确保容器执行完成后不再重启；
- kubelet：在Pod失效时重启，不论将RestartPolicy设置为何值，也不会对Pod进行健康检查。

</br></br>

### k8s创建一个pod流程

- 客户端提交Pod的配置信息（可以是yaml文件定义好的信息）到kube-apiserver；
- Apiserver收到指令后，通知给controller-manager创建一个资源对象；
- Controller-manager通过api-server将pod的配置信息存储到ETCD数据中心中；
- Kube-scheduler检测到pod信息会开始调度预选，会先过滤掉不符合Pod资源配置要求的节点，然后开始调度调优，主要是挑选出更适合运行pod的节点，然后将pod的资源配置单发送到node节点上的kubelet组件上。
- Kubelet根据scheduler发来的资源配置单运行pod，运行成功后，将pod的运行信息返回给scheduler，scheduler将返回的pod运行状况的信息存储到etcd数据中心。

</br></br>

### 删除pod流程

Kube-apiserver会接受到用户的删除指令，默认有30秒时间等待优雅退出，超过30秒会被标记为死亡状态，此时Pod的状态Terminating，kubelet看到pod标记为Terminating就开始了关闭Pod的工作；

关闭流程如下：

- pod从service的endpoint列表中被移除；
- 如果该pod定义了一个停止前的钩子，其会在pod内部被调用，停止钩子一般定义了如何优雅的结束进程；
- 进程被发送TERM信号（kill -14）
- 当超过优雅退出的时间后，Pod中的所有进程都会被发送SIGKILL信号（kill -9）。

</br></br>

### Service是什么

Pod每次重启或者重新部署，其IP地址都会产生变化，这使得pod间通信和pod与外部通信变得困难，这时候，就需要Service为pod提供一个固定的入口。

Service的Endpoint列表通常绑定了一组相同配置的pod，通过负载均衡的方式把外界请求分配到多个pod上。

</br></br>

### 怎么服务注册

Pod启动后会加载当前环境所有Service信息，以便不同Pod根据Service名进行通信。

</br></br>

### K8S集群外流量怎么访问Pod

可以通过Service的NodePort方式访问，会在所有节点监听同一个端口，比如：30000，访问节点的流量会被重定向到对应的Service上面。

</br></br>

### k8s数据持久化的方式有哪些

**1）EmptyDir（空目录）：**

没有指定要挂载宿主机上的某个目录，直接由Pod内保部映射到宿主机上。类似于docker中的manager volume。

**主要使用场景：**

- 只需要临时将数据保存在磁盘上，比如在合并/排序算法中；
- 作为两个容器的共享存储，使得第一个内容管理的容器可以将生成的数据存入其中，同时由同一个webserver容器对外提供这些页面。

**emptyDir的特性：**

同个pod里面的不同容器，共享同一个持久化目录，当pod节点删除时，volume的数据也会被删除。如果仅仅是容器被销毁，pod还在，则不会影响volume中的数据。

总结来说：emptyDir的数据持久化的生命周期和使用的pod一致。一般是作为临时存储使用。

**2）Hostpath：**

将宿主机上已存在的目录或文件挂载到容器内部。类似于docker中的bind mount挂载方式。

这种数据持久化方式，运用场景不多，因为它增加了pod与节点之间的耦合。

一般对于k8s集群本身的数据持久化和docker本身的数据持久化会使用这种方式，可以自行参考apiService的yaml文件，位于：/etc/kubernetes/main…目录下。

**3）PersistentVolume（简称PV）：**

基于NFS服务的PV，也可以基于GFS的PV。它的作用是统一数据持久化目录，方便管理。

在一个PV的yaml文件中，可以对其配置PV的大小，指定PV的访问模式：

- ReadWriteOnce：只能以读写的方式挂载到单个节点；
- ReadOnlyMany：能以只读的方式挂载到多个节点；
- ReadWriteMany：能以读写的方式挂载到多个节点。以及指定pv的回收策略：
- recycle：清除PV的数据，然后自动回收；
- Retain：需要手动回收；
- delete：删除云存储资源，云存储专用；

> PS：这里的回收策略指的是在PV被删除后，在这个PV下所存储的源文件是否删除）。

若需使用PV，那么还有一个重要的概念：PVC，PVC是向PV申请应用所需的容量大小，K8s集群中可能会有多个PV，PVC和PV若要关联，其定义的访问模式必须一致。定义的storageClassName也必须一致，若群集中存在相同的（名字、访问模式都一致）两个PV，那么PVC会选择向它所需容量接近的PV去申请，或者随机申请。

</br></br>

### 常用命令

- kubectl create：创建资源(kubectl create -f docker-registry.yaml)
- kubectl run：快速创建容器(kubectl run nginx --image=nginx)
- kubectl get：获取资源列表(kubectl get pods)
- kubectl describe：查看资源详细信息(kubectl describe pods/nginx)
- kubectl exec：在容器内执行命令(kubectl exec podName -c containerName -i -t -- bash -il)
- kubectl logs：查看 pod 日志(kubectl logs nginx)
- kubectl delete：删除一个资源(kubectl delete pods --all)

</br></br>

### 简述Helm及其优势

Helm是Kubernetes的软件包管理工具。类似Ubuntu中使用的APT、CentOS中使用的yum 或者Python中的 pip 一样。

Helm能够将一组Kubernetes资源打包统一管理, 是查找、共享和使用为Kubernetes构建的软件的最佳方式。

Helm中通常每个包称为一个Chart，一个Chart是一个目录（一般情况下会将目录进行打包压缩，形成name-version.tgz格式的单一文件，方便传输和存储）。

在Kubernetes中部署一个可以使用的应用，需要涉及到很多的 Kubernetes 资源的共同协作。使用Helm则具有如下优势：

- 统一管理、配置和更新这些分散的Kubernetes的应用资源文件；
- 分发和复用一套应用模板；
- 将应用的一系列资源当做一个软件包管理。
- 对于应用发布者而言，可以通过 Helm 打包应用、管理应用依赖关系、管理应用版本并发布应用到软件仓库。


对于使用者而言，使用Helm后不用需要编写复杂的应用部署文件，可以以简单的方式在Kubernetes上查找、安装、升级、回滚、卸载应用程序。

</br></br>



### 参考链接

- https://dockone.io/article/2434304
- https://juejin.cn/post/7070507980952698911
- https://www.modb.pro/db/99717