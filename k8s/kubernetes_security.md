

## API Server认证管理

Kubernetes集群中所有资源的访问和变更都是通过Kubernetes API Server的REST API实现的， 所以集群安全的关键点就在于如何识别并认证客户端身份（Authentication） ， 以及随后访问权限的授权（Authorization） 这两个关键问题。

Kubernetes集群有两种用户账号： 第1种是集群内部的ServiceAccount； 第2种是外部的用户账号， 可能是某个运维人员或外部应用的账号。Kubernetes并不支持常规的个人账号， 但拥有被Kubernetes集群的CA证书签名的有效证书， 个人用户就可被授权访问Kubernetes集群了，在这种情况下， 证书中Subject（主题） 里的信息被当作用户名如"/CN=bob"。 因此， 任一Kubernetes API的访问都属于以下三种方式之一：

- 以证书方式访问的普通用户或进程， 包括运维人员及kubectl、kubelets等进程
- 以Service Account方式访问的Kubernetes的内部服务进程
- 以匿名方式访问的进程。

## API Server授权管理

当客户端发起API Server调用时， API Server内部要先进行用户认证， 然后执行用户授权流程， 即通过授权策略（Authorization Policy）决定一个API调用是否合法。 对合法用户进行授权并随后在用户访问时进行鉴权， 是权限与安全系统中的重要一环。 简单地说， 授权就是授予不同的用户不同的访问权限。 API Server目前支持以下授权策略。

- AlwaysDeny： 表示拒绝所有请求， 仅用于测试。(1.3版本废弃)
- AlwaysAllow： 允许接收所有请求， 如果集群不需要授权流程， 则可以采用该策略。
- ABAC（Attribute-Based Access Control） ： 基于属性的访问控制， 表示使用用户配置的授权规则对用户的请求进行匹配和控制。
- RBAC： Role-Based Access Control， 是基于角色的访问控制。
- Webhook： 通过调用外部的REST服务对用户进行授权。
- Node： 是一种对kubelet进行授权的特殊模式

Node授权策略用于对kubelet发出的请求进行访问控制， 与用户的应用授权无关， 属于Kubernetes自身安全的增强功能。 简单来说， 就是限制每个Node只访问它自身运行的Pod及相关的Service、 Endpoints等信息； 也只能受限于修改自身Node的一些信息， 比如Label； 也不能操作其他Node上的资源。 而之前用RBAC这种通用权限模型其实并不能满足
Node这种特殊的安全要求， 所以将其剥离出来定义为新的Node授权策略。

### ABAC授权模式详解

在API Server启用ABAC模式时， 集群管理员需要指定授权策略文件的路径和名称（--authorization-policy-file=SOME_FILENAME） ， 授权策略文件里的每一行都以一个Map类型的JSON对象进行设置， 它被 称为“访问策略对象”。 在授权策略文件中， 集群管理员需要设置访问策略对象中的apiVersion、 kind、 spec属性来确定具体的授权策略， 其中，apiVersion的当前版本为abac.authorization.kubernetes.io/v1beta1； kind被设置为Policy； spec指详细的策略设置， 包括主体属性、 资源属性、 非资源属性这三个字段。

*对主体属性说明如下*

- user（用户名）：字符串类型， 该字符串类型的用户名来源于Token文件（--token-auth-file参数设置的文件） 或基本认证文件中用户名称段的值
- group（ 用户组） ： 在被设置为“system： authenticated”时， 表示匹配所有已认证请求； 在被设置为“system： unauthenticated”时， 表示匹配所有未认证请求。

*对资源属性说明如下*

- apiGroup（API组） ： 字符串类型， 表明匹配哪些API Group，例如extensions或*（表示匹配所有API Group） 。
- namespace（命名空间） ： 字符串类型， 表明该策略允许访问某个Namespace的资源， 例如kube-system或*（表示匹配所有Namespace）。
- resource（资源） ： 字符串类型， 表明该策略允许访问某个资源， 例如pods或*（表示匹配所有资源）。

*对非资源属性说明如下*

- nonResourcePath（非资源对象类路径） ： 非资源对象类的URL路径， 例如/version或/apis， *表示匹配所有非资源对象类的请求路径，也可以将其设置为子路径， /foo/*表示匹配所有/foo路径下的所有子路径。
- readonly（ 只读标识） ： 布尔类型， 当它的值为true时， 表明仅允许GET请求通过。

#### ABAC授权算法

API Server进行ABAC授权的算法为： 在API Server收到请求之后，首先识别出请求携带的策略对象的属性， 然后根据在策略文件中定义的策略对这些属性进行逐条匹配， 以判定是否允许授权。 如果有至少一条匹配成功， 这个请求就通过了授权。

- 要允许所有认证用户做某件事， 则可以写一个策略， 将group属性设置为system： authenticated。
- 要允许所有未认证用户做某件事， 则可以把策略的group属性设置为system： unauthenticated。
- 要允许一个用户做任何事， 则将策略的apiGroup、 namespace、resource和nonResourcePath属性设置为“*”即可。

#### 使用kubectl时的授权机制

kubectl使用API Server的/api和/apis端点来获取版本信息。 要验证kubectl create/update命令发送给服务器的对象， kubectl则需要向OpenAPI查询， 对应的URL路径为/openapi/v2。

使用ABAC授权模式时， 以下特殊资源必须显式地通过nonResourcePath属性设置。

- API版本协商过程中的/api、 /api/*、 /apis和/apis/*。
- 通过kubectl version命令从服务器中获取版本时的/version。
- create/update操作过程中的/swaggerapi/*。

#### 对Service Account进行授权

Service Account会自动生成一个ABAC用户名（username）， 用户名按照以下命名规则生成：

```bash
system:serviceaccount:<namespace>:<serviceaccountname>
```

创建新命名空间时会产生一个如下名称的Service Account

```bash
system:serviceaccount:<namespace>:default
```

如果希望kube-system命名空间下的Service Account "default" 具有所有权限，就需要再策略文件中加入如下内容

```bash
apiVersion: abac.authorization.kubernetes.io/v1beta1
kind: Policy
spec:
  user: system:serviceaccount:kube-system:default
  apiGroup: "*"
  namespace: "*"
  resource: "*"
  nonResourcePath: "*"
```

### RBAC授权模式详解

要使用RBAC授权模式， 首先需要在kube-apiserver服务的启动参数authorization-mode（ 授权模式） 的列表中加上RBAC， 例如--authorization-mode=...， RBAC。

- RBAC的API资源对象说明

在RBAC管理体系中， Kubernetes引入了4个资源对象： Role、ClusterRole、 RoleBinding和ClusterRoleBinding。

![[image-2025-02-10-09-42-06-446.png]]

1. 角色（ Role） 和集群角色（ClusterRole）

    一个角色就是一组权限的集合， 在Role中设置的权限都是许可 （Permissive）形式的， 不可以设置拒绝（Deny） 形式的规则。 Role设置的权限将会局限于命名空间（namespace） 范围内， 如果需要在集群级别设置权限， 就需要使用ClusterRole了。

2. 角色绑定（ RoleBinding） 和集群角色绑定（ClusterRoleBinding）

    角色绑定或集群角色绑定用来把一个角色绑定到一个目标主体上，绑定目标可以是User（用户） 、 Group（组） 或者Service Account。RoleBinding用于某个命名空间中的授权， ClusterRoleBinding用于集群范围内的授权。

在集群角色绑定（ClusterRoleBinding） 中引用的角色只能是集群级别的角色（ClusterRole） ， 而不能是命名空间级别的Role。一旦通过创建RoleBinding或ClusterRoleBinding与某个Role或ClusterRole完成了绑定， 用户就无法修改与之绑定的Role或ClusterRole了。 只有删除了RoleBinding或ClusterRoleBinding， 才能修改Role或ClusterRole。

### Pod安全策略

为了更精细地控制Pod启动或更新时的安全管理， Kubernetes从1.5版本开始引入PodSecurityPolicy资源对象对Pod安全策略进行管理， 到1.18版本时达到Beta阶段。 通过对PodSecurityPolicy的设置， 管理员可以控制Pod的运行条件， 以及可以使用系统的哪些功能。 PodSecurityPolicy是集群范围内的资源对象， 不属于命名空间范围。

若需要启用PodSecurityPolicy机制， 则首先需要设置kube-apiserver服务的启动参数--enable-admission-plugins来开启PodSecurityPolicy准入控制器：

```bash
--enable-admission-plugins=···, PodSecurityPolicy
```

> 注意， 在开启PodSecurityPolicy准入控制器后， 系统中还没有任何PodSecurityPolicy策略配置时， Kubernetes默认不允许创建任何Pod， 需要管理员创建适合的PodSecurityPolicy策略和相应的RBAC授权策略，
Pod才能创建成功。












