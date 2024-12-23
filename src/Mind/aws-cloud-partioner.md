# aws cloud practitioner essentials
## chapter1 介绍 aws
### 核心
- 总结 aws 的好处
- 描述按需交付和云部署的不同
- 总结现收现付的定价模式
### 云定义（aws）：云计算是在互联网上按需交付IT资源，按需付费
### 云计算交付模型：
- 三类：云、本地、混合
- 云： 
  - 所有应用都在云上
  - 迁移已有的应用到云上
  - 在云上设计建设新应用
- 本地（也成为私有云）：
  - 通过虚拟化和资源管理工具部署资源。
  - 通过应用程序管理和虚拟化技术提高资源利用率。
- 混合：
  - 将基于云的资源连接到本地基础设施。
  - 将基于云的资源与遗留IT应用程序集成。
### 云计算优势：
- 将前期费用换成可变费用
- 受益于大规模的规模经济（规模扩大成本降低）
- 停止耗费金钱用于运行和维护数据中心
- 停止猜测容量
- 提高速度和敏捷
- 全球化部署

## chapter2 云计算
### EC2（Elastic Compute Cloud）
EC2：在云中提供安全、可调整大小的计算容量
- 优势
  - 启动时间短
  - 工作结束后可以关闭
  - 仅在工作时间支付费用
  - 您可以通过仅为您需要或想要的服务器容量付费来节省成本。
- 类型
  - 普通型：适配场景（应用服务、游戏服务、后端、中小数据库）
  - 计算优化型：计算优化实例对于受益于高性能处理器的计算绑定应用程序非常理想。与通用实例一样，您可以为web、应用程序和游戏服务器等工作负载使用计算优化实例。
  - 内存优化型：内存优化实例旨在为在内存中处理大型数据集的工作负载提供快速性能。在计算中，内存是一个临时存储区域。它保存了中央处理器（CPU）完成操作所需的所有数据和指令。在计算机程序或应用程序能够运行之前，它是从存储器加载到内存中的。这个预加载过程使CPU可以直接访问计算机程序。
  - 加速计算：加速计算实例使用硬件加速器或协处理器来比在cpu上运行的软件更有效地执行某些功能。这些函数的示例包括浮点数计算、图形处理和数据模式匹配。
  - 存储优化型：存储优化实例是为需要对本地存储上的大型数据集进行高顺序读写访问的工作负载而设计的。适合存储优化实例的工作负载示例包括分布式文件系统、数据仓库应用程序和高频在线事务处理（OLTP）系统。
- 实例保留： 1 year or 3 years

### EC2 价格
- 灵活（on-demand）
  按需实例的示例用例包括开发和测试应用程序，以及运行具有不可预测使用模式的应用程序。对于持续一年或更长时间的工作负载，不建议使用按需实例，因为这些工作负载可以使用保留实例节省更多的成本。
  按需实例非常适合不能中断的短期不规则工作负载。没有前期费用或最低合同适用。这些实例持续运行，直到您停止它们，并且您只需为所使用的计算时间付费。
- 预留实例（Reserved Instances）：保留实例是应用于在您的帐户中使用按需实例的计费折扣。保留实例有两种类型：
  - 标准保留实例
    如果清楚需要的配置和运行的区域保留实例要求说明以下条件：
    - 实例型号，比如m5.large
    - 操作系统
    - 租户：专有或默认租户
  - 可转换保留实例
    如果您需要在不同的可用区或不同的实例类型中运行EC2实例，那么可转换保留实例可能适合您。注意：当您需要运行EC2实例的灵活性时，您可以获得更大的折扣。
    在保留实例期限结束时，您可以继续使用Amazon EC2实例而不会中断。但是，在您完成以下其中一项之前，您将按需收费：
    - 终止实例
    - 购买与实例属性（实例族和大小、区域、平台和租赁）匹配的新Reserved Instance。
- EC2实例节省计划
- 竞价实例
  Spot实例非常适合具有灵活的开始和结束时间或可以承受中断的工作负载。现货实例使用未使用的Amazon EC2计算能力，并为您提供高达按需价格90%的成本节约。
- 专用主机
  专用主机是具有Amazon EC2实例容量的物理服务器，完全专用于您的使用。

### EC2扩展
- EC2 可伸缩性：包括从您需要的资源开始，并设计您的体系结构，以便通过向外或向内扩展来自动响应不断变化的需求。因此，您只需为所使用的资源付费。您不必担心缺乏满足客户需求的计算能力。AWS 提供 Amazon EC2 Auto Scaling 服务实现。
- Amazon EC2 Auto Scaling 能力
  - 动态扩展响应不断变化的需求。
  - 预测性扩展会根据预测的需求自动调度正确数量的Amazon EC2实例。

### 使用弹性负载均衡（ELB elastic load balance）引导流量
- ELB:弹性负载平衡是一项AWS服务，它可以跨多个资源（如Amazon EC2实例）自动分配传入的应用程序流量(ELB 是 regin 级别)。
- 负载均衡器充当所有传入web流量到您的自动缩放组的单点联系人。这意味着，当您根据传入流量添加或删除Amazon EC2实例时，这些请求将首先路由到负载平衡器。然后，请求分散到将处理它们的多个资源。例如，如果您有多个Amazon EC2实例，弹性负载平衡会在多个实例之间分配工作负载，这样就不需要单个实例承担大部分工作负载。
- 尽管Elastic Load Balancing和Amazon EC2 Auto Scaling是独立的服务，但它们协同工作以确保在Amazon EC2中运行的应用程序能够提供高性能和可用性。

### 消息和队列（MQ）
- 单体应用：应用程序由多个组件组成。组件之间相互通信以传输数据、完成请求并保持应用程序运行。假设您有一个具有紧密耦合组件的应用程序。这些组件可能包括数据库、服务器、用户界面、业务逻辑等等。这种类型的体系结构可以看作是一个整体应用程序。
  - 优点：所有代码都在一个代码库中，便于开发和维护，简化部署
  - 缺点：在这种应用程序体系结构方法中，如果单个组件失败，其他组件也会失败，甚至可能导致整个应用程序失败。
- 微服务：应用程序组件是松散耦合的。在这种情况下，如果单个组件失败，其他组件将继续工作，因为它们彼此通信。松耦合可以防止整个应用程序失败。
在AWS上设计应用程序时，您可以采用微服务方法，使用实现不同功能的服务和组件。两种服务促进了应用程序集成：Amazon Simple Notification Service （Amazon SNS）和Amazon Simple Queue Service （Amazon SQS）。
- 松耦合架构：单点故障不会造成级联故障
- Amazon Simple Notification Service（SNS）
  - 是一种发布/订阅服务。发布者使用Amazon SNS主题向订阅者发布消息。这与咖啡店类似；收银员向调制咖啡的咖啡师提供咖啡订单。
  - 在Amazon SNS中，订阅者可以是web服务器、电子邮件地址、AWS Lambda函数或其他几个选项。
- Amazon Simple Queue Service （SQS）：发送、存储、接收消息（任意大小）在不同的组件中
  - playload: data contained with message

### 其他计算服务
- serverless：你不可以看到或者接触到底层基础设置
- AWS Lambda: 它允许您运行代码，而无需配置或管理服务器。
  - 在使用AWS Lambda时，您只需为所消耗的计算时间付费。费用仅在代码运行时应用。您还可以为几乎任何类型的应用程序或后端服务运行代码，而无需任何管理。
  - 使用场景短时间运行函数、服务型应用、事件驱动应用、不需要配置服务器
- ECS （elastic container service）：是一个高度可扩展的高性能容器管理系统，使您能够在AWS上运行和扩展容器化应用程序。
  - 运行在 EC2上，docker 管理容器
  - AWS支持使用开源的Docker社区版和基于订阅的Docker企业版。使用Amazon ECS，您可以使用API调用来启动和停止启用docker的应用程序。
- EKS （elastic kubernetes service）运行在 EC2上，是一个完全托管的服务，您可以使用它在AWS上运行Kubernetes。
- aws Fragate (serverless for ECS or EKS) 是一个用于容器的无服务器计算引擎。它兼容Amazon ECS和Amazon EKS。
  - 使用AWS Fargate时，您不需要配置或管理服务器。AWS Fargate为您管理服务器基础设施。您可以更专注于创新和开发应用程序，并且只需为运行容器所需的资源付费。

## chapter3 全球基础设施和可靠性
- 学习内容
  - 总结 aws 全球基础设施的优势
  - 描述可用区基本概念
  - 描述 Amazon CloudFront 和 edge locations 的优势
  - 比较提供AWS服务的不同方法。
- AWS 全球基础设施
  - AWS Regions
    - 选择 Region 考虑的点
      - 法律法规
      - 跟客户的距离
      - 功能可用性：不同区域提供的服务不尽相同
      - 价格
  - Availability Zones（AZ）
    可用分区是一个区域内的单个数据中心或一组数据中心。可用区彼此相距几十英里。这足够接近可用区之间的低延迟（请求和接收内容之间的时间）。但是，如果灾难发生在区域的一部分，它们之间的距离足够远，可以减少多个可用区受到影响的可能性。
- edge location （边缘节点）
  - AWS cloudFront （CDN aws 版）使用它来存储离客户更近的内容缓存副本，以便更快地交付。
  - Amazon Route 53 （DNS）
  - AWS outposts 在自己的数据中心运行 aws （私有云）
- 如何使用 AWS 资源
  - AWS managment console AWS管理控制台是一个基于web的接口，用于访问和管理AWS服务。您可以快速访问最近使用的服务，并通过名称、关键字或首字母缩略词搜索其他服务。控制台包括向导和自动化工作流，可以简化完成任务的过程。
  - AWS command line interface（cli）AWS CLI使您能够直接从一个工具中的命令行控制多个AWS服务。AWS CLI支持Windows、macOS和Linux三种操作系统。通过使用AWS CLI，您可以自动执行服务和应用程序通过脚本执行的操作。例如，您可以使用命令启动Amazon EC2实例，将Amazon EC2实例连接到特定的Auto Scaling组，等等。
  - AWS software developer kits（SDK）访问和管理AWS服务的另一个选择是软件开发工具包（sdk）。sdk使您可以通过为您的编程语言或平台设计的API更轻松地使用AWS服务。sdk使您能够将AWS服务与现有应用程序一起使用，或创建将在AWS上运行的全新应用程序。为了帮助您开始使用sdk， AWS为每种受支持的编程语言提供了文档和示例代码。支持的编程语言包括c++、Java、。net等。
  - AWS Elastic Beanstalk 您提供代码和配置设置，然后Elastic Beanstalk部署执行以下任务所需的资源：
    - 调整能力
    - 负载平衡
    - 自动缩放
    - 应用程序运行状况监视
  - AWS CloudFormation 使用AWS CloudFormation，您可以将基础设施视为代码。这意味着您可以通过编写几行代码来构建环境，而不是使用AWS Management Console单独提供资源。AWS CloudFormation以安全、可重复的方式提供资源，使您能够频繁地构建基础架构和应用程序，而无需执行手动操作。它决定在管理堆栈时执行正确的操作，并在检测到错误时自动回滚更改。

## chapter4 网络
- 学习内容
  - 描述网络基本概念
  - 描述共有网络和私有网络的区别
  - 用现实生活场景解释一个虚拟专用网关。
  - 使用现实生活场景解释虚拟专用网（VPN）。
  - 描述 AWS Direct Connect 的优势
  - 描述混合部署的好处
  - 描述IT策略中使用的安全层
  - 描述客户用于与AWS全球网络交互的服务。
- 连接 AWS
  - Amazon Virtual Private Cloud (Amazon VPC) 可以用来围绕AWS资源建立边界的网络服务是Amazon Virtual Private Cloud (Amazon VPC)“Amazon VPC”提供AWS云的隔离部分。在这个独立的部分中，您可以在您定义的虚拟网络中启动资源。在VPC中，您可以将资源划分为不同的子网。子网是VPC的一部分，可以包含Amazon EC2实例等资源。
  - Internet gateway 因特网网关 当公网流量需要访问VPC时，需要为VPC添加外网网关。互联网网关是VPC和互联网之间的连接。你可以想到一个互联网网关,就像客户使用进入咖啡店的门口一样。如果没有互联网网关,没有人可以访问您的VPC中的资源。
  - Virtual private gateway 虚拟专用网关: 当需要访问VPC中的私有资源时，可以使用虚拟私有网关。虚拟的私有网关使您能够在您的VPC和一个私有网络之间建立一个虚拟的私有网络(VPN)连接,例如一个实时的数据中心或内部的企业网络。一个虚拟的私人网关,只有当它来自一个被批准的网络时,才允许使用VPC。
  - AWS Direct Connect 是一种服务，可以让您在数据中心和VPC之间建立专用私有连接。AWS Direct Connect提供的专用连接可帮助您降低网络成本并增加可以通过网络传输的带宽量。
- 子网和网络权限控制列表（ACL）
  - 子网（subnets）：子网是 VPC一部分，用户可以根据安全需求或业务需求对 VPC 内资源进行分组，子网分为公共子网和私有子网
    - 公共子网（public subnets）：包含需要被公开访问的资源，比如在线存储网站
    - 私有子网（private subnets）：应该仅通过私有网络进行访问的资源，比如数据库或者私人信息和历史记录
    - 同一个 VPC 的私有子网可以和公共子网进行通信
  - VPC内网络流量
    - 当一个用户访问在 AWS cloud 的应用是，请求发送一个包（packet）。包是一个数据单元通过因特网或网络时。
    - 网络访问控制列表：VPC的一部分用于检查包文是否允许进入子网
  - 网络 ACLs
    - 网络访问控制列表（网络 ACLs）是一个虚拟的防火墙用于控制流量进出子网
    - 每一个 AWS 账户都包含一个默认的网络ACL。当配置自己的 VPC时可以使用默认的 网络 ACL或者自定义 网络 ACL
  - 无状态包过滤（stateless packet filtering）
    - 网络 ACL 进行无状态包过滤，它们什么都不记住，并检查每一个穿越子网边界的数据包，进站和出站
  - 安全组（security groups）
    - 安全组是一个虚拟防火墙用于控制进站和出站 Amazon EC2 实例的流量
    - 默认，安全组拒绝所有你入的网络流量，运行所有出去的流量。你可以添加自定义规则配置，其他流量将会被拒绝
    - 如果你有多个 EC2 实例在同一个 VPC，你可以为每一个实例进行配置相同的安全组或不同的安全组
  - 状态包过滤（stateful packet filtering）
    - 安全组进行有状态包过滤，它们记得之前对传入数据包所做的决定
    - 当请求数据包响应返回到实体时，安全组会记录之前的请求，安全组允许响应继续进行，而不考虑入站安全组规则
- 全球网络
  - DNS（domain name system）域名翻译成 IP 地址
  - Amazon Route 53
    DNS web 服务
    路由规则：
    * 简单路由策略 - 对于为您的域执行给定功能的单一资源（例如为 example.com 网站提供内容的 Web 服务器），可以使用该策略。在私有托管区域中，可以使用简单的路由创建记录。
    * 故障转移路由策略 - 如果您想要配置主动-被动故障转移，则可以使用该策略。在私有托管区域中，可以使用失效转移路由创建记录。
    * 地理位置路由策略 - 如果您想要根据用户的位置来路由流量，则可以使用该策略。在私有托管区域中，可以使用地理位置路由创建记录。
    * 地理位置临近度路由策略 - 用于根据资源的位置来路由流量，以及（可选）将流量从一个位置中的资源转移到另一个位置中的资源。您可以使用邻近地理位置路由在私有托管区域中创建记录。
    * 延迟路由策略-当您有多个资源 AWS 区域 并且想要将流量路由到延迟最佳的区域时使用。在私有托管区域中，可以使用延迟路由创建记录。
    * 基于 IP 的路由策略 – 如果您希望根据用户的位置来路由流量，并且获得流量来源的 IP 地址，则可以使用该策略。
    * 多值答案路由策略 — 当您希望 Route 53 响应随机选择最多八条健康记录的DNS查询时使用。在私有托管区域中，可以使用多值应答路由创建记录。
    * 加权路由策略 - 用于按照您指定的比例将流量路由到多个资源。在私有托管区域中，可以使用加权路由创建记录。

## chapter5 存储和数据库
- 学习内容
  - 总结存储和数据库的概念
  - 描述 aws EBS 的优势
  - 描述 aws S3（simple storage service）的优势
  - 描述 aws EFS（elastic file system）的优势
  - 总结各种存储方案
  - 描述 AWS RDS（Relational Database Service）的优势
  - 描述 AWS DynamoDB 的优势
  - 总结各种数据库方案
- EBS
  - 实例存储（instance block）
    块级存储卷的行为类似于物理硬盘驱动器
    实例存储（在新选项卡中打开）为Amazon EC2实例提供临时块级存储。实例存储是物理地连接到EC2实例的主机计算机的磁盘存储，因此具有与实例相同的生命周期。当实例终止时，您将丢失实例存储中的所有数据。
    要创建EBS卷，需要定义配置（例如卷大小和类型）并提供它。创建EBS卷之后，它可以附加到Amazon EC2实例。
    因为EBS卷用于需要持久化的数据，所以备份数据非常重要。可以通过创建
  - EBS 快照（snapshots）
    是一种增量备份。这意味着对卷进行的第一次备份将复制所有数据。对于后续备份，仅保存自最近快照以来发生更改的数据块。
- S3
  - 对象存储
    在对象存储中，每个对象包括（数据、元数据、key）
    数据可以是图像、视频、文本文档或任何其他类型的文件。
    元数据包含有关数据是什么、如何使用数据、对象大小等信息。
    对象的键是它的唯一标识符。
  - S3
    S3服务提供对象级别的存储。Amazon S3将数据作为对象存储在桶中。
    您可以将任何类型的文件上传到Amazon S3，例如图像、视频、文本文件等。例如，您可以使用Amazon S3来存储备份文件、网站的媒体文件或归档文档。Amazon S3提供无限的存储空间。Amazon S3中一个对象的最大文件大小为5tb。
    当您将文件上传到Amazon S3时，您可以设置权限来控制文件的可见性和访问权限。您还可以使用Amazon S3版本控制功能跟踪对象随时间的变化。
  - S3 存储类
    使用Amazon S3，您只需为使用的内容付费。您可以从一系列存储类（在新选项卡中打开）中选择适合您的业务和成本需求的存储类。在选择Amazon S3存储类时，请考虑以下两个因素：
    * 您计划多久检索一次数据
    * 你需要您的数据可用性如何
    



