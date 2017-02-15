#DCOS
## 什么是DC/OS
DC/OS是基于Apache Mesos分布式系统内核的分布式操作系统。它使得能够管理多个机器，就像它们是单个计算机一样。它自动化资源管理，安排进程布局，促进进程间通信，并简化分布式服务的安装和管理。其包含的Web界面和可用的令行界面（CLI）便于远程管理和监控群集及其服务。

当你操作数据中心时，有一组常用的操作。在单个计算机环境中，您的操作系统自动处理这些事情。就像在你的笔记本上运行程序一样，你从来不用关心你的程序由哪个内核去运行，这都是系统自动去执行的。

在笔记本电脑上，将应用程序调度到特定核心的工作由内核处理。抽象这个过程，内核管理您的计算机上的所有资源，包括内存，磁盘，输入和输出等。没有理由内核必须只在一台计算机上运行。在多台计算机上管理资源的程序可以解决同样的问题。事实上，这是一个容器编排器。它可以像整个数据中心的组合资源的内核，而不是单个计算机的资源。

在这一点上，在某些资源上运行应用程序的行为被自动化。不幸的是，这只是一个小块的谜题。我们的现代操作系统包含大量的软件，用于解决许多其他常见的计算问题，并允许我们专注于做实际工作，而不是管理我们的计算机。DC / OS包括这些常见问题的解决方案，而在您自己研究解决方案之前，手动将它们集成到数据中心的计算机中。

### 通用管理接口 - GUI和CLI
现在不会没有GUI和CLI的计算机，我们不再在穿孔卡上编写程序了。没有理由我们应该与低级API，甚至RESTful交互，除非我们做一些高级的功能扩展的时候。
在某些时候，变得不可能作为个人与数据中心中的每个计算机进行交互。对于纯粹的理性，您必须在顶部分层抽象，并查看整个数据中心。你需要一个单一的地方来看看正在运行和运行的地方。您可以轻松地监视和调试问题，减少停机时间。没有理由SSH到数百个主机只是为了找到特定应用程序的日志。
CLI与GUI一样重要，有时不能给予它到期的信用。常规任务可以完全跳过GUI，甚至可以集成到您的正常环境中，例如VI或Emacs。对于特殊情况，CLI最终是一个很好的工具，脚本和自动化那些你不想担心的事情。

### 服务发现
写一个独立的服务并且不需要联系任何其他服务是很少的。对于三层应用程序，您需要找出如何配置中间层以联系后端数据库。在单个主机上，这可以通过端口完成。所有你需要做的是指定一个端口，假设IP地址是localhost，你准备好了。不幸的是，一旦你移动到多个主机，会出现一些问题：
		运行容器的主机IP是什么？
		如何配置客户端以连接到正确的主机IP？
		容器重新安排后，如何动态更新客户端以连接到新的主机IP？
		如何管理两个应用程序要绑定到像8080这样的常用端口？
DNS是解决方案的开始。它最初是为了在互联网的规模照顾服务发现。不幸的是，它不能很好地在一个动态改变IP的世界，因为缓存问题，一般不能解决端口。
您可以为客户端提供与容器通信的虚拟IP和端口。这种架构在内部和在每个云中工作，并将构建应用程序带回单个计算机的世界。传统和现代应用程序依赖这个简单的解决方案来完成他们的业务，而不需要任何代码更改。

### 软件包管理
应用程序往往由多个没有配置的进程组成。微服务可以由几百个进程组成，所有进程都有自己的配置。采取类似这样的方式，并将其部署在数据中心是一个耗时，微妙的过程。很容易在您的环境中误解特定配置设置的含义，或者无法使应用程序运行。
了解所有这些复杂性的一种方法是创建一个您希望在数据中心中利用的常见应用程序的包。有许多选项，如数据库和消息队列，你建立在之上。通过将进程和配置捆绑在一个包中，您可以快速开始，并确保您遵循最佳做法。

## 架构
操作系统抽象化CPU，RAM和网络等资源，并为应用程序提供通用服务。DC/OS是一个分布式操作系统，它抽象机器群集的资源并提供公共服务。这些公共服务包括跨多个节点运行进程，服务发现和包管理。本主题讨论DC/OS的架构及其组件的交互。

为了简化对DC/OS的理解，我们将重用Linux内核和用户空间的术语。内核空间是用户无法访问的保护区，涉及低级操作，如资源分配，安全性和进程隔离。用户空间是用户应用程序和高阶服务所在的位置，例如操作系统的GUI。

### 架构图解
DC/OS内核空间由Mesos主机和Mesos代理组成。用户空间包括系统组件，例如Mesos-DNS，分布式DNS代理和诸如Marathon或Spark等服务。用户空间还包括由服务管理的进程，例如Marathon应用程序。
![](https://docs.mesosphere.com/wp-content/uploads/2016/06/dcos-architecture-100000ft-4.png)
在我们深入介绍不同DC / OS组件之间的交互细节之前，让我们定义所使用的术语。
		
* Mesos Master:聚合来自所有代理节点的资源提供，并将它们提供给注册的框架。
* Scheduler程序:服务的调度程序组件，例如Marathon调度程序
* 	用户：也称为客户端，是集群内部或外部的启动过程的应用程序，例如提交Marathon应用程序规范的人类用户。
* Mesos Agent:代表一个框架运行一个离散的Mesos任务。它是向Mesos主机注册的Agent程序实例。Agent节点的同义词是工作节点或从节点。您可以拥有私人或公共代理节点
* Executor：在Agent节点上启动以运行服务的任务。
* Task：由Mesos框架计划并在Mesos Agent上执行的工作单元。
* 	Process:由客户端发起的任务的逻辑集合，例如Marathon应用程序或Metronome作业。

#### Kernel Space
在DC / OS中，内核空间管理跨集群的资源分配和两级调度。内核空间中的两种类型的进程是Mesos masters和agents:

* Mesos masters:`mesos-master`节点该进程协调在Mesos代理上运行的任务。Mesos主进程从Mesos代理接收资源报告，并将这些资源分配给注册的DC / OS服务，如Marathon或Spark。当主导Mesos主服务器由于崩溃或脱机升级失败时，备用Mesos主控器自动成为领导者而不中断正在运行的服务。动物园管理员执行领导选举

* Mesos agents: `mesos-agent`节点代表框架运行离散Mesos任务。私有Agent节点通过不可路由的网络运行部署的应用和服务。公共Agent节点在可公开访问的网络中运行DC / OS应用和服务。`mesos-slave`Mesos Agent上的进程管理其本地资源（CPU核心，RAM等），并将这些资源注册到Mesos Master。它还接受来自Mesos Master的调度请求，并调用Executor通过容器化器启动任务： 
	*  Mesos容器化程序使用特定于Linux的功能（如cgroups和命名空间）提供轻量级的容器化和执行者的资源隔离。
	* Docker容器化程序支持启动包含Docker镜像的任务。

#### User Space 
DC/OS用户空间跨越系统服务和用户服务，如Chronos或Kafka。

-  默认情况下，系统服务已安装并在DC / OS集群中运行，并包括以下内容
	* 管理路由器是一个开放源代码的NGINX配置，为DC / OS服务提供中心认证和代理
	* Exhibitor在安装过程中自动配置ZooKeeper，并为ZooKeeper提供一个可用的Web UI
	* Mesos-DNS提供服务发现，允许应用和服务通过使用域名系统（DNS）找到彼此。
	* Minuteman是内部层4负载均衡器。
	* 分布式DNS代理是内部DNS分派器
	* 	DC / OS Marathon是用于DC / OS的“init系统”的本机Marathon实例，用于启动和监视DC / OS服务。
	*  ZooKeeper，一种管理DC/OS服务的高性能协调服务。
- 用户服务
	* 	DC / OS中的用户服务由调度程序（负责代表用户调度任务）和执行程序（在Agent程序上运行任务）组成。
	*  用户级应用程序，例如通过Marathon启动的NGINX网络服务器。
	
### Distributed process management
本节介绍DC / OS集群中进程的管理，从资源分配到进程的执行。
在高级别，当用户启动进程时，在DC / OS组件之间发生这种交互。通信发生在不同层之间，例如用户与调度器交互，以及在层内，例如Mesos agent与Mesos master的通信。

![](https://docs.mesosphere.com/wp-content/uploads/2016/06/dcos-architecture-distributed-process-management-concept-4-800x504@2x.png)
这里是一个例子，使用Marathon服务和用户启动基于Docker镜像的容器：

![](https://docs.mesosphere.com/wp-content/uploads/2016/06/dcos-architecture-distributed-process-management-example-5-800x458@2x.png)

上述组件之间的时间相互作用看起来像这样。注意，Executors和Task被折叠成一个块，因为在实践中经常是这样。
![](https://docs.mesosphere.com/wp-content/uploads/2016/06/dcos-architecture-distributed-process-management-seq-diagram-4-1200x726@2x.png)

具体来说，这里是步骤：

1.  客户机/调度程序初始化：客户机需要知道如何连接到调度程序以启动一个进程，例如通过Mesos-DNS或DC/OS CLI。
2. Mesos主机向调度器发送资源提供：资源提供基于通过Agent管理的集群资源和Mesos主机中的DRF算法
3. 调度程序拒绝资源提供，因为没有来自客户端的进程请求挂起。只要没有客户端启动进程，调度程序将拒绝来自主服务器的提议。
4. 客户端启动进程启动。例如，这可以是通过DC / OS 服务选项卡或通过HTTP端点创建Marathon应用程序的用户`/v2/app`。
5. Mesos Master 发起资源邀约：例如:cpus(*):1; mem(*):128; ports(*):[21452-21452]
6. 如果资源邀约匹配调度程序对进程的要求，则它接受该邀约并向Mesos Master 发送 launchTask请求。
7. Mesos Master 指派Mesos Agent 去launch task
8. Mesos  Agent通过executor执行task
9. Executor向Agent报告task执行情况
10. Mesos Agent 将任务执行情况报告给Mesos Master
11. Mesos Master 将任务执行情况报告给Scheduler
12. Scheduler向客户端报告任务执行情况

## 特性
以下是DC/OS一些特性的汇总和概述

* 高资源利用率
* 混合工作负载共置
* 容器编排
* 可扩展资源隔离
* 状态存储支持
* 公共和私有包存储库
* 云无关安装程序
* Web和命令行界面
* 弹性可扩展性
* 高可用性
* 零停机升级
* 集成测试组件
* 服务发现和分布式负载平衡
* 分布式负载平衡器的控制和管理平面
* 集群周边安全
* 身份和访问管理服务
* 具有LDAP，SAML和OpenID Connect的外部身份提供者
* 具有加密通信的群集安全性
* 使用容器级别授权的工作负载隔离
* 每个具有可扩展虚拟网络（SDN）的容器IP
* 虚拟网络子网的网络隔离

### 高资源利用率

DC / OS可以轻松充分利用您的计算资源。

决定在何处运行进程以最好地利用集群资源是很困难的，事实上是NP-hard。决定在哪里放置长期运行的服务，随着时间的推移资源需求不断变化甚至更困难。实际上，没有一个调度程序可以高效和有效地放置所有类型的任务。没有任何方法可以无限地配置单个调度程序，普遍可移植，闪电快速，易于使用 - 所有在同一时间。

DC / OS通过将资源管理与任务调度分离来管理此问题。Mesos管理CPU，内存，磁盘和GPU资源。任务放置委托给更高级别的调度员，这些调度员更了解他们的任务的具体要求和约束。这种称为两级调度的模型使得能够有效地共同定位多个工作负载。
### 混合工作负载共置
DC / OS可以在同一硬件上轻松运行所有计算任务。

对于调度长期运行的服务，DC / OS与Marathon紧密集成，为启动微服务，Web应用程序或其他调度程序提供了一个坚实的阶段。

对于其他类型的工作，DC / OS可以轻松地从工业标准调度程序库中选择和安装。这打开了运行批处理作业，分析管道，消息队列，大数据存储等的门户。
对于复杂的自定义工作负载，您甚至可以编写自己的调度程序来优化和精确控制特定任务的调度逻辑。

### 容器编排
DC / OS提供易于使用的容器编排，开箱即用。

Docker提供了巨大的开发经验，但是试图在生产中运行Docker容器提出了重大的挑战。为了克服这些挑战，DC / OS包括Marathon作为核心组件，为您提供一个生产级，战斗性的调度程序，能够协调集装式和非集装式工作负载。

使用Marathon，您有能力达到极端规模，在数千个节点上安排数万个任务。您可以使用高度可配置的声明性应用程序定义来强制使用节点，集群和分组亲和性的高级布局约束。

### 可扩展资源隔离
DC / OS可以配置多个资源隔离区。

并非所有任务都有相同的要求。一些需要最大隔离以确保安全或性能。其他是短暂的，公共的，或容易重新启动。大多数是在之间的某个地方。

最简单的隔离方法是将Agent委托给Docker。在DC / OS上运行Docker容器很简单，但是当涉及隔离时，Docker是一个钝器。Mesos containerizer更加灵活，有多个可独立配置的隔离器和可插拔的定制隔离器。Mesos containerizer甚至可以运行Docker容器而不被链接到脆弱性dockerd。

### 支持状态存储
DC / OS为您的服务提供多个持久和临时存储选项。

外部持久性卷，bread和butter块存储，在许多平台上可用。这些是易于使用和因为他们的工作就像传统的服务器磁盘，但是，通过设计，它们牺牲了弹性和复制的速度。

分布式文件系统是云本机应用程序的主要部分，但它们往往需要以新的方式思考存储，并且由于基于网络的交互几乎总是较慢。

本地临时存储是Mesos默认为将临时磁盘空间分配给服务。这对于许多无状态或半无状态的12因子和云本地应用程序来说是足够的，但是对于状态服务可能不够好。
本地持久卷弥补了间隙，并提供快速，持久的存储。如果您的服务已在复制数据或您的驱动器是RAID并备份到近线或磁带驱动器，本地卷可能会给您足够的容错，而没有速度税。

### 公共和私有包存储库
DC / OS可以轻松安装社区和专有包装服务。

Mesosphere Universe包存储库将您与开放源代码行业标准调度程序，服务和应用程序库连接在一起。为什么要重新发明轮子，如果你不必？利用社区项目来处理批处理作业调度，高可用性数据存储，强大的消息队列等。

DC / OS还支持从多个软件包存储库安装：您可以托管您自己的私有软件包，以在您的公司或客户中共享。

### 云无关安装程序
DC / OS安装程序使在任何物理或虚拟机群集上轻松安装DC / OS。

对于具有自己的内部部署硬件或虚拟机配置基础架构的用户，GUI或CLI安装程序提供了一种快速直观的安装DC / OS的方法。

对于部署到公共云的用户，DC / OS为AWS，Azure和Packet提供了多个可配置的云配置模板。

对于高级用户，高级安装程序提供了一个可脚本化的自动化界面，以便与您喜欢的配置管理系统集成。

### Web和命令行界面
DC / OS Web和命令行界面使其易于监视和管理群集及其服务。

DC / OS Web界面允许您通过直观的基于浏览器的导航，实时图形和交互式调试
工具来监视资源分配，运行服务，当前任务，组件运行状况，可用软件包等。

DC / OS命令行界面从终端的舒适性提供对DC / OS的控制。它是强大的，但易于脚本化，与方便的插件与已安装的服务交互。

### 弹性可扩展性
DC / OS让您能够轻松地随着拨号盘的转动而上下调节服务。

只要您的服务支持，水平缩放在Marathon中是微不足道的。您可以随时更改服务实例的数量。DC / OS甚至可以使用Marathon负载平衡器根据会话数自动调整实例数。

Marathon也支持垂直缩放，允许您为服务分配更多或更少的资源，并自动执行滚动更新以重新计划实例，而不会停机。

将节点添加到DC / OS群集也是一个快照。DC / OS安装程序使用不可变工件，允许您配置新节点，而不必重新编译，重新配置或从轻量级远程存储库重新下载组件包。

### 高可用性
DC / OS高度可用，使您的服务也很容易使用。
任务关键型服务需要对其自身以及其运行的平台和基础设施进行健康监控，自我修复和容错。DC / OS为您提供多层保护。

为了实现自我修复，DC / OS服务由Marathon监控，并在失败时重新启动。甚至不支持分发或复制的传统服务也可以由Marathon自动重新启动，以最大限度地延长正常运行时间并减少服务中断。此外，所有核心DC / OS组件（包括Marathon）都由DC / OS诊断服务监视，并systemd在其发生故障时重新启动。
为了实现容错，DC / OS可以在多个主配置中运行。这不仅提供系统级容错，而且提供调度程序级容错。DC / OS甚至可以在升级期间生存节点故障，而不会丢失服务。

### 零停机升级
DC / OS提供自动化更新服务和系统，具有零停机时间。

在Marathon上运行的DC / OS服务可以使用滚动，蓝绿色或金丝雀部署模式进行更新。如果更新失败，请单击鼠标将其回滚。这些强大的工具对于最小化停机时间和用户中断至关重要。

DC / OS本身还通过其强大的安装程序支持零停机升级。通过单个组合更新，保持最新的最新开源组件。

### 集成测试组件
DC / OS提供了一套经过良好测试的开源组件，并将它们与单个组合安装程序一起烘焙。

混合和匹配开源组件可能是一种痛苦。你永远不知道哪些版本将一起工作或他们的互动的副作用将是什么。让Mesos专家为您处理它！快速进入生产，专注于您的产品的质量，而不是您的平台的稳定性。

### 服务发现和分布式负载平衡
DC / OS包括用于自动化服务发现和负载平衡的几个选项。

分布式服务创建分布式问题，但你不必自己解决它们。DC / OS包括自动DNS端点生成，用于服务查找的API，用于高速内部通信的传输层（L4）虚拟IP代理，以及用于面向外部服务的应用层（L7）负载平衡。

### 分布式负载均衡器的控制和管理平面
企业DC / OS提供集中的管理和控制平面，用于服务可用性和性能监控。

虽然分布式负载平衡器是DC / OS服务的服务发现和服务可用性的理想选择，但是监视和管理它们需要工具和工作。企业DC / OS带有用于DC / OS分布式负载均衡器的集中式控制和管理平面，其中包括将所有分布式引擎统一为单个服务中心视图和单一服务运行状况度量标准的聚合API。企业DC / OS还包括一个服务性能和健康监控UI，可帮助监控服务性能以及根本原因服务降级问题，并识别根本原因。

### 集群周边安全
DC / OS提供了规定性设计，以确保DC / OS群集与任何客户端（UI /浏览器，CLI，API客户端等）之间通过管理安全区域进行管理和编程通信，并且所有请求都通过SSL安全通道传输。DC / OS主节点是管理安全区域内DC / OS群集的入口点，更具体地说，API网关是一个名为“Admin Router”的组件，用作管理到DC / OS群集的所有管理连接的反向代理。

### 身份和访问管理服务
企业DC / OS包括内置的身份和访问管理服务，允许我们的用户创建用户和组，并为每个用户和组分配不同级别的授权权限。Enterprise DC / OS支持以下类型的用户和组：

* 本地用户
* 本地组
* 远程LDAP用户
* 远程LDAP组（仅用于导入到本地组）
* 远程SAML用户
* 服务用户帐户

企业DC / OS IAM服务还包括对可以分配给上述每个主体/用户的授权控制的支持。从DC / OS 1.8开始，用户可以被赋予一组特定的权限，“主题”可以对“对象”执行“动作”，其中“对象”可以是到马拉松应用程序的特定DC / OS服务的API端点组和“操作”枚举对象上可能的一组操作，例如“创建，读取，更新或删除”。

### 具有LDAP，SAML和OpenID Connect的外部身份提供者

企业DC / OS集成了支持LDAP v3接口（包括Microsoft Active Directory）和基于SAML的身份提供程序的身份提供程序，以便您可以从现有用户目录导入DC / OS外部用户，并管理DC内用户和用户组的授权/ OS。

### 具有加密通信的群集安全性
企业DC / OS旨在安全地在内部和云中运行。为了确保集群安全性，Enterprise DC / OS支持DC / OS Cluster内部组件之间的加密通信。这是通过确保DC / OS运行与证书颁发机构的DC / OS主节点的证书，并且所有代理节点在引导时安装CA.crt来实现。此机制确保DC / OS群集内的各种服务之间的所有通信都通过安全SSL通道。

### 使用容器级别授权的工作负载隔离
企业DC / OS支持细粒度工作负载隔离，以允许组织中的多个业务组在共享集群内运行容器和工作负载，但仍然保证除了在变化的工作负载之间由Linux cGroup提供的性能隔离之外，还有安全隔离。工作负载安全隔离由DC / OS授权模块执行，该模块在每个代理节点上运行，负责对DC / OS IAM服务进行授权检查，以验证工作负载的用户/所有者是否被授权执行他们尝试的操作以在群集（包括代理节点）中的任何位置执行。

### 每个具有可扩展虚拟网络（SDN）的容器IP
DC / OS内置支持使用容器网络接口（CNI）标准的虚拟网络。默认情况下，创建一个名为“dos”的虚拟网络，并且任何附加到虚拟网络的容器都会接收到自己的专用IP。这允许用户运行不适合动态分配的端口的工作负载，而是绑定现有应用程序配置中的现有端口。现在，通过支持专用IP /容器，工作负载可以自由绑定到任何端口，因为每个容器都可以访问整个可用的端口范围。

### 虚拟网络子网的网络隔离
DC / OS现在支持在安装时创建多个虚拟网络，并将不重叠的子网与每个虚拟网络相关联。此外，DC / OS用户可以跨DC / OS Agent节点编程网络隔离规则，以确保跨虚拟网络子网的流量隔离。

## Components
DC / OS由许多单独的开源组件组成，这些组件精确配置为一起工作。
您可以登录到DC / OS集群中的任何主机，并通过检查目录来查看当前正在运行的服务/etc/systemd/system/dcos.target.wants/。
您可以在packages目录中的https://github.com/dcos/dcos/ repo中查看DC / OS组件详细信息

Component| Description
------- | -------
Admin Router Agent | 这个组件（`dcos-adminrouter-agent`) is a high performance web server and a reverse proxy server that lists all of the agent nodes in your cluster.
Admin Router Master | 此组件是高性能Web服务器和反向代理服务器，其列出群集中的所有主节点。
Admin Router Reloader | 此组件`dcos-adminrouter-reload.service`重新启动管理路由器Nginx服务器，以便它可以获取新的DNS解析，例如master.mesos和leader.mesos。
Admin Router Reloader Time | 此组件每小时一次`dcos-adminrouter-reload.timer`设置管理路由器重新加载器间隔。
Admin Router Service|此组件是由Mesosphere创建的开源Nginx配置，为集群中的DC / OS服务提供集中身份验证和代理。管理路由器服务`dcos-adminrouter.service`是DC / OS的核心内部负载均衡器。管理路由器是定制的Nginx，代理端口上的所有内部服务80。
Certificate Authority|此组件`dcos-ca.service`是DC / OS证书颁发机构功能。有关详细信息，请参阅文档。
Cluster ID|cluster-id服务为每个集群生成通用唯一标识符（UUID）。我们使用此ID远程跟踪群集运行状况（如果已启用）。这种远程跟踪允许我们的支持团队更好地协助我们的客户。
Diagnostics|此组件`dcos-3dt.service`是DC / OS systemd组件的诊断实用程序。此服务在每个主机上运行， 跟踪systemd单元的内部状态。服务以两种模式运行，包含或不包含`-pull`参数。如果在主主机上运行， 它将执行/`opt/mesosphere/bin/3dt -pull`哪些查询Mesos-DNS查找集群中已知主设备的列表，然后查询主设备（通常是主设备）:5050/statesummary并获取代理列表.从此群集主机的完整列表中，它查询所有3DT运行状况端点:`1050/system/health/v1/health`。此端点返回该主机上的DC / OS systemd单元的运行状况。主3DT进程，以及这种聚合也暴露/system/health/v1/端点，以通过`unit`或馈送此数据node到DC / OS用户界面。
Diagnostics socket|此组件`dcos-3dt.socket`是DC / OS分布式诊断工具主API和聚合套接字。
DNS Dispatcher|此组件`dcos-spartan.service`是符合RFC5625的DNS转发器。它的作用是双重调度DNS到多个上游解析器，并根据一些规则将DNS路由到上游或Mesos DNS。
DNS Dispatcher Watchdog|此组件（`dcos-spartan-watchdog.service`）确保DNS调度程序正在运行且运行正常。如果DNS调度程序不正常，此看门狗服务将终止它。
DNS Dispatcher Watchdog Timer|此组件`dcos-spartan-watchdog.timer`每5分钟唤醒一次DNS Dispatcher Watchdog，以查看DC / OS是否需要重新启动DNS调度程序。
Downloads Service|这个组件（`dcos-download.service`) downloads the DC/OS installation tarball on first boot.
Erlang Port Mapping Daemon|此组件`dcos-epmd.service`支持称为Minuteman的内部DC / OS层4负载均衡器
Exhibitor|这个组件`dcos-exhibitor.service`是Zookeeper的参展商主管。DC / OS使用来自Netflix的项目“Exhibitor”来管理和自动化ZooKeeper的部署。
Generate resolv.conf|此组件`dcos-gen-resolvconf.service`动态地设置/etc/resolv.conf，以便每个群集主机可以使用Mesos-DNS来将任务名称解析为IP和端口地址。
Generate resolv.conf Timer|此组件dcos-gen-resolvconf.timer定期更新systemd解析为Mesos DNS
History Service|此组件`dcos-history-service.service`使DC / OS UI能够显示集群使用统计信息，并存储UI的仪表板图形数据。此数据存储在磁盘上24小时。与存储此数据一起，历史服务还公开用于DC / OS用户界面查询的HTTP API。所有涉及内存，CPU和磁盘使用的DC / OS群集统计信息都由此服务驱动。如果无法访问历史记录服务，则每次用户在UI中打开“仪表板”选项卡时，UI都将重新启动图表时间轴。通过访问历史记录服务，用户界面将在图表时间轴中显示历史数据，最多60秒。
Identity and Access Management|此组件`dcos-bouncer.service`是DC / OS身份和访问管理功能。有关详细信息，请参阅文档。
Job|此组件`dcos-metronome.service`驱动DC/OS作业。有关详细信息，请参阅文档。
Layer 4 Load Balancer|此组件`dcos-minuteman.service`（也称为Minuteman）是支持多层微服务架构的DC / OS第4层负载均衡器。有关详细信息，请参阅文档。
Logrotate Mesos Master|此组件`dcos-logrotate-master.service`自动管理Mesos主进程的日志文件的压缩，删除和邮件发送。这确保DC / OS服务不会使磁盘上的日志数据过多的群集主机过载。
Logrotate Mesos Slave|此组件`dcos-logrotate-agent.service`/自动为Agent节点旋转Mesos代理日志文件。
Logrotate Timer|这些组件`dcos-logrotate-agent.timer`并`dcos-logrotate-master.timer`设置logrotate间隔为2分钟。
Marathon|该组件`dcos-marathon.service`是DC / OS Marathon实例，用于启动和监视DC / OS应用程序和服
Mesos Agent|此组件`dcos-mesos-slave.service`是专用代理节点的mesos 从属进程。
Mesos Agent Public|此组件`dcos-mesos-slave-public.service`是公有的Agent 节点
Mesos DNS|此组件`dcos-mesos-dns.service`在集群中提供服务发现。Mesos-DNS是`dcos-mesos-dns.service `DC / OS群集的内部DNS服务。Mesos-DNS为$service.mesos所有集群主机提供命名空间。例如，您可以使用域名登录到您的leading mesos主机`ssh leader.mesos`。
Mesos Master|此组件`dcos-mesos-master.service`是协调代理任务的mesos-master进程。
Mesos Persistent Volume Discovery|此组件`dcos-vol-discovery-pub-agent.service`在安装期间连接到Agent节点上的现有Mesos卷安装。有关Mesos Persistent Volumes的更多信息，请参阅文档。
Network Metrics Aggregator|此组件`dcos-networking_api.service`提供DC / OS网络指标聚合服务和API。有关详细信息，请参阅文档。
Package service|此组件`dcos-cosmos.service`是内部打包API服务。每次dcos package install从CLI 运行时都会访问此服务。此API将DC / OS程序包从DC / OS Universe部署到DC / OS群集。
REX-Ray|此组件（`dcos-rexray.service`）是用于在Marathon中启用外部持久卷的REX-Ray存储方法。
System Package Manager API|此组件`dcos-pkgpanda-api.service`提供了用于安装和卸载DC / OS组件的API。
System Package Manager API socket|此组件`dcos-pkgpanda-api.socket`是系统包管理器API套接字。
Secrets Service|此组件`dcos-secrets.service`保护重要值，如私钥，凭据和数据库密码。有关详细信息，请参阅文档
Signal|此组件`dcos-signal.service`定期向Mesosphere返回高级集群信息，以帮助改进DC / OS，并提供对集群问题的高级监控。信号查询/system/health/v1/report主导主机上的诊断服务端点，并将此数据发送到SegmentIO以用于跟踪度量和客户支持。
Signal Timer|此组件每小时一次`dcos-signal.timer`设置信号分量间隔。
Vault|此组件`dcos-vault.service`是Secrets组件的存储后端。它管理提交给秘secrets件的secret的安全和持久处理。
Virtual Network Service|此组件`dcos-navstar.service`是一个提供虚拟网络和DNS服务的守护程序。它是网络覆盖协调器。有关详细信息，请参阅文档。

