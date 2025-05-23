

Kubernetes网络模型设计的一个基础原则是： 每个Pod都拥有一个独立的IP地址， 并假定所有Pod都在一个可以直接连通的、 扁平的网络空间中。 所以不管它们是否运行在同一个Node（宿主机） 中， 都要求它们可以直接通过对方的IP进行访问。

在Kubernetes世界里， IP是以Pod为单位进行分配的。 一个Pod内部的所有容器共享一个网络堆栈（相当于一个网络命名空间， 它们的IP地址、 网络设备、 配置等都是共享的） 。 按照这个网络原则抽象出来的为每个Pod都设置一个IP地址的模型也被称作IP-per-Pod模型。

为每个Pod都设置一个IP地址的模型还有另外一层含义， 那就是同一个Pod内的不同容器会共享同一个网络命名空间， 也就是同一个Linux网络协议栈。 这就意味着同一个Pod内的容器可以通过localhost连接对方的端口。 这种关系和同一个VM内的进程之间的关系是一样的， 看起来Pod内容器之间的隔离性减小了， 而且Pod内不同容器之间的端口是共享的， 就没有所谓的私有端口的概念了。 如果你的应用必须使用一些特定的端口范围， 那么你也可以为这些应用单独创建一些Pod。 反之， 对那些没有特殊需要的应用， 由于Pod内的容器是共享部分资源的， 所以可以通过共享资源相互通信， 这显然更加容易和高效。

Kubernetes集群至少应该包含三个网络，如图网络环境所示。一个是各主机（Master、Node和etcd等）自身所属的网
络，其地址配置于主机的网络接口，用于各主机之间的通信，例如，Master与各Node之间的通信。此地址配置于Kubernetes集群构建之前，它并不能由Kubernetes管理，管理员需要于集群构建之前自行确定其地址配置及管理方式。第二个是Kubernetes集群上专用于Pod资源对象的网络，它是一个虚拟网络，用于为各Pod对象设定IP地址等网络参数，其
地址配置于Pod中容器的网络接口之上。Pod网络需要借助kubenet插件或CNI插件实现，该插件可独立部署于Kubernetes集群之外，亦可托管于Kubernetes之上，它需要在构建Kubernetes集群时由管理员进行定义，而后在创建Pod对象时由其自动完成各网络参数的动态配置。第三个是专用于Service资源对象的网络，它也是一个虚拟网络，用于为Kubernetes集群之中的Service配置IP地址，但此地址并不配置于任何主机或容器的网络接口之上，而是通过Node之上的kube-proxy配置为iptables或ipvs规则，从而将发往此地址的所有流量调度至其后端的各Pod对象之上。Service网络在Kubernetes集群创建时予以指定，而各Service的地址则在用户创建Service时予以动态配置。

![[image-2025-01-27-01-07-54-828.png]]


## Docker网络基础

Docker技术依赖于近年来Linux内核虚拟化技术的发展， 所以Docker对Linux内核有很强的依赖。 Docker使用到的技术有网络命名空间（ Network Namespace） 、 Veth设备对、 网桥、 ipatables和路由。

### 网络命名空间

为了支持网络协议栈的多个实例， Linux在网络栈中引入了网络命名空间， 这些独立的协议栈被隔离到不同的命名空间中。 处于不同命名空间中的网络栈是完全隔离的， 彼此之间无法通信。 通过对网络资源的隔离， 就能在一个宿主机上虚拟多个不同的网络环境。 Docker正是利用了网络的命名空间特性， 实现了不同容器之间的网络隔离。

由于网络命名空间代表的是一个独立的协议栈， 所以它们之间是相互隔离的， 彼此无法通信， 在协议栈内部都看不到对方。 那么有没有办法打破这种限制， 让处于不同命名空间中的网络相互通信，甚至与外部的网络进行通信呢？ 答案是“有， 应用Veth设备对即可”。Veth设备对的一个重要作用就是打通了相互看不到的协议栈之间的壁垒， 它就像一条管子， 一端连着这个网络命名空间的协议栈， 一端连着另一个网络命名空间的协议栈。 所以如果想在两个命名空间之间通信，就必须有一个Veth设备对。 后面会介绍如何操作Veth设备对来打通不同命名空间之间的网络。

```bash
# 创建一个网络命名空间
ip netns add <name>
# 删除一个网络命名空间
ip netns del <name>
# 在一个网络命名空间中添加一个网卡
ip link add <name> type veth peer name <name>
# 将网卡添加到网络命名空间中
ip link set <name> netns <name>
# 在网络命名空间中删除网卡
ip link del <name>
# 在网络命名空间中查看网卡
ip netns exec <name> ip link
# 在网络命名空间中查看路由
ip netns exec <name> ip route
# 在网络命名空间中运行命令
ip netns exec <name> <command>
# 如果需要执行多个命令，可以先试用bash进入到内部的shell界面，然后再执行命令
ip netns exec <name> bash
# 退出到外部网络命名空间
exit
```

### Veth设备对

引入Veth设备对是为了在不同的网络命名空间之间通信， 利用它可以直接将两个网络命名空间连接起来。 由于要连接两个网络命名空间，所以Veth设备都是成对出现的， 很像一对以太网卡， 并且中间有一根直连的网线。 既然是一对网卡， 那么我们将其中一端称为另一端的peer。在Veth设备的一端发送数据时， 它会将数据直接发送到另一端， 并触发
另一端的接收操作。

```bash
# 创建一个Veth设备对
ip link add <name> type veth peer name <name>
# 使用ip link show 查看所有网络接口
ip link show
# 删除一个Veth设备对
ip link del <name>
# 将其中一个网卡添加到网络命名空间中
ip link set <veth1> netns <netns1>
# 查看网络设备是否转移成功
ip netns exec <netns1> ip link show
# 为veth设备对添加IP地址，这样才能进行通讯
ip netns exec netns1 ip addr add 10.1.1.1/24 dev veth1
ip netns exec netns2 ip addr add 10.1.1.2/24 dev veth2
ip addr add 10.1.1.3/24 dev veth3
# 拉起网卡
ip netns exec netns1 ip link set dev veth1 up
ip netns exec netns2 ip link set dev veth2 up
ip link set dev veth3 up
# 启动网卡
ip link set <veth1> up
```

### Docker的网络实现

标准的Docker支持以下4类网络模式:

- host模式：使用 `--net=host` 指定。
- bridge模式：使用 `--net=bridge` 指定，为默认设置。
- none模式：使用 `--net=none` 指定，不创建任何网络。
- container模式：使用 `--net=container:<name|id>` 指定，将容器的网络设置与指定容器的网络设置相同。

在Kubernetes管理模式下通常只会使用bridge模式

在bridge模式下， Docker Daemon首次启动时会创建一个虚拟网桥，默认的名称是docker0， 然后按照RPC1918的模型在私有网络空间中给这个网桥分配一个子网。 针对由Docker创建的每一个容器， 都会创建一个虚拟以太网设备（Veth设备对） ， 其中一端关联到网桥上， 另一端使用Linux的网络命名空间技术映射到容器内的eth0设备， 然后在网桥的地址段内给eth0接口分配一个IP地址。

## Kubernetes的网络实现

- 容器到容器之间的直接通信。
- 抽象的Pod到Pod之间的通信。
- Pod到Service之间的通信。
- 集群内部与外部组件之间的通信

### 容器到容器的通信

同一个Pod内的容器（Pod内的容器是不会跨宿主机的） 共享同一个网络命名空间， 共享同一个Linux协议栈。 所以对于网络的各类操作，就和它们在同一台机器上一样， 它们甚至可以用localhost地址访问彼此的端口。

这么做的结果是简单、 安全和高效， 也能减少将已存在的程序从物理机或者虚拟机中移植到容器下运行的难度。

### Pod之间的通信

每一个Pod都有一个真实的全局IP地址， 同一个Node内的不同Pod之间可以直接采用对方Pod的IP地址通信， 而且不需要采用其他发现机制， 例如DNS、 Consul或者etcd。

Pod容器既有可能在同一个Node上运行， 也有可能在不同的Node上运行， 所以通信也分为两类： 同一个Node上Pod之间的通信和不同Node上Pod之间的通信。

## CNI网络模型

- 容器运行时必须在调用任意插件前为容器创建一个新的网络命名空间
- 容器运行时必须确定此容器所归属的网络（一个或多个） ， 以及每个网络必须执行哪个插件。
- 容器运行时必须按照先后顺序为每个网络运行插件将容器添加到每个网络中
- 容器生命周期结束后， 容器运行时必须以反向顺序（相对于添加容器执行顺序） 执行插件， 以使容器与网络断开连接。
- 容器运行时一定不能为同一个容器的调用执行并行（parallel）操作， 但可以为多个不同容器的调用执行并行操作
- 容器运行时必须对容器的ADD和DEL操作设置顺序， 以使得ADD操作最终跟随相应的DEL操作。 DEL操作后面可能会有其他DEL操作， 但插件应自由处理多个DEL操作（即多个DEL操作应该是幂等的）。
- 容器必须由ContainerID进行唯一标识。 存储状态的插件应使用联合主键（ network name、 CNI_CONTAINERID、 CNI_IFNAME） 进行存储

## Calico插件的原理和部署示例

Calico是一个基于BGP的纯三层的网络方案， 与OpenStack、Kubernetes、 AWS、 GCE等云平台都能够良好地集成。 Calico在每个计算节点都利用Linux Kernel实现了一个高效的vRouter来负责数据转发。每个vRouter都通过BGP1协议把在本节点上运行的容器的路由信息向整个Calico网络广播， 并自动设置到达其他节点的路由转发规则。 Calico保证所有容器之间的数据流量都是通过IP路由的方式完成互联互通的。Calico节点组网时可以直接利用数据中心的网络结构（L2或者L3） ， 不需要额外的NAT、 隧道或者Overlay Network， 没有额外的封包解包， 能
够节约CPU运算， 提高网络效率。

### Calico的主要组件如下

- Felix： Calico Agent， 运行在每个Node上， 负责为容器设置网络资源（IP地址、 路由规则、 iptables规则等） ， 保证跨主机容器网络互通
- etcd： Calico使用的后端存储
- BGP Client： 负责把Felix在各Node上设置的路由信息通过BGP广播到Calico网络。
- Route Reflector： 通过一个或者多个BGP Route Reflector完成大规模集群的分级路由分发。
- CalicoCtl： Calico命令行管理工具。

## k8s的网络策略

为了实现细粒度的容器间网络访问隔离策略， Kubernetes从1.3版本开始引入了Network Policy机制， 到1.8版本升级为networking.k8s.io/v1稳定版本。 Network Policy的主要功能是对Pod或者Namespace之间的网络通信进行限制和准入控制， 设置方式为将目标对象的Label作为查询条件， 设置允许访问或禁止访问的客户端Pod列表。 目前查询条件可以作用于Pod和Namespace级别。

为了使用Network Policy， Kubernetes引入了一个新的资源对象NetworkPolicy， 供用户设置Pod之间的网络访问策略。 但这个资源对象配置的仅仅是策略规则， 还需要一个策略控制器（Policy Controller） 进行策略规则的具体实现。 策略控制器由第三方网络组件提供， 目前Calico、 Cilium、 Kube-router、 Romana、 Weave Net等开源项目均支持网络策略的实现。

网络策略的设置主要用于对目标Pod的网络访问进行控制， 在默认情况下对所有Pod都是允许访问的， 在设置了指向Pod的NetworkPolicy网络策略后， 到Pod的访问才会被限制。

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  labels:
    app.kubernetes.io/component: grafana
    app.kubernetes.io/name: grafana
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 11.4.0
  name: grafana
  namespace: monitoring
spec:
  egress:
  - {}
  # 定义允许访问目标Pod的入站白名单规则， 满足from
  #条件的客户端才能访问ports定义的目标Pod端口号。
  ingress:
#  对符合条件的客户端Pod进行网络放行， 规则包括基于客
#  户端Pod的Label、 基于客户端Pod所在命名空间的Label或者客户端的IP
#  范围
  - from:
    - podSelector:
        matchLabels:
          app.kubernetes.io/name: prometheus
    # 允许访问的目标Pod监听的端口号
    ports:
    - port: 3000
      protocol: TCP
  # 定义该网络策略所针对的Pod，这里选择包含以下标签的Pod，也就是这个网络策略作用到那个Pod身上
  podSelector:
    matchLabels:
      app.kubernetes.io/component: grafana
      app.kubernetes.io/name: grafana
      app.kubernetes.io/part-of: kube-prometheus
  # 网络策略类型，包含ingress 和 egress ，用于设置目标Pod的入站和出站的网络限制。 如果未指定policyTypes， 则系统默认会设置Ingress类型若设置了egress策略， 则系统自动设置Egress类型
  policyTypes:
  - Egress
  - Ingress
```

### 为命名空间配置默认的网络策略

在一个命名空间没有设置任何网络策略的情况下， 对其中Pod的ingress和egress网络流量并不会有任何限制。 在命名空间级别可以设置一些默认的全局网络策略， 以便管理员对整个命名空间进行统一的网络策略设置。








